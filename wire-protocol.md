# Introduction

The Multiplexer wire protocol is designed to be as simple as possible to implement, while
still having reasonable performance. I'm aiming for it to be possible to
make ad-hoc implementations for simple cases, with 0 dependencies.
This means there's not currently any bit packing. In fact, every individual
piece of information fits exactly in a single byte. This isn't ideal for
efficiency, but does make things much easier to reason about. It does lead to
real limitations, however. For example, you can only have 256 simultaneous
streams in a single Multiplexer, and have to include logic to reuse the ids that close.
It might be better to use a larger number for the stream id, but the bigger you make it
the more overhead of the protocol, unless you do a bunch of fancy bit packing with
variable-sized fields (a la
[varints](https://developers.google.com/protocol-buffers/docs/encoding)).
I'm not strictly opposed to this, but I suspect it would
be omnistreams2. I'm more interested in simplicity for now.


# Protocol 

Everything in omnistreams happens in discrete messages, where each message has
a known size and guaranteed in-order delivery. All interfaces operate on
byte streams. Most languages provide easy mechanisms for converting things
like JSON messages to a byte array. If you're sending an actual byte stream,
you need to break it up into a series of chunks.

Each message is prepended with a single byte message type. The types are:


```
MESSAGE_TYPE_CONTROL_MESSAGE       = 0
// From initiating side
MESSAGE_TYPE_CREATE_RECEIVE_STREAM = 1
MESSAGE_TYPE_STREAM_DATA           = 2
MESSAGE_TYPE_STREAM_END            = 3
MESSAGE_TYPE_CANCEL_RECEIVE_STREAM = 4
// From accepting side
MESSAGE_TYPE_STREAM_REQUEST_DATA   = 5
MESSAGE_TYPE_CANCEL_SEND_STREAM    = 6
```

All messages except control messages use the second byte as the stream ID.
The initiator determines stream IDs.

Here are the semantics for each type:

## Generic messages

```
MESSAGE_TYPE_CONTROL_MESSAGE
```

Control messages allow a convenient out-of-band communication channel for
the Multiplexer, Control messages are not tied to any specific stream.
Like stream creation metadata, control messages operate on byte arrays and
the application is responsible for encoding/decoding.

## Initiator messages

```
MESSAGE_TYPE_CREATE_RECEIVE_STREAM 
```

Instructs the remote end to create the producer side of a new conduit. The
local end handles the consumer side. Any value in the range 0-255 can be used
for the stream ID, but the local side is responsible for ensuring data messages
are correctly mapped to ids.

The rest of the message after the first 2 bytes is metadata. The application is
responsible for encoding/decoding, but implementations are encouraged to
provide helper functions for at least one common format (such as JSON).

```
MESSAGE_TYPE_STREAM_DATA
```

Send a chunk of data for the given stream ID. If the remote side recieves
a data chunk for a stream it doesn't recognize, it should ignore the data
and immediately send a `MESSAGE_TYPE_CANCEL_SEND_STREAM`.

The local side shall not send more messages than the remote side has indicated
it's ready for via `MESSAGE_TYPE_STREAM_REQUEST_DATA`.

```
MESSAGE_TYPE_STREAM_END
```

Indicates the natural end of data for the given stream. No data is sent with
this message, just the message type and stream id. Therefore, all data should
already have been sent via `MESSAGE_TYPE_STREAM_DATA` messages.

```
MESSAGE_TYPE_CANCEL_RECEIVE_STREAM
```

Instructs the acceptor end that the stream is being canceled, and to destroy
any knowledge of the stream, and cease any processing for that stream.


## Acceptor messages

```
MESSAGE_TYPE_STREAM_REQUEST_DATA
```

Indicates to the remote side that the local side is ready to receive more
data for the stream. This is used to implement flow control/backpressuring.

The third byte of the message indicates how many additional messages the local
side is ready to receive. For things like JSON messages, it's one-to-one. If
you're sending a byte stream, you'll need to have some sort of pre-negotiated
concept of chunk size. This can be sent in the metadata when the stream is
created.

The requested amount should be viewed as how much data you
*can* handle, not a demand for a specific amount. It's common for a stream
to naturally end while there is still a lot of outlying request. This simple
scheme can be used to simulate higher level concepts. For example, when a
stream is first opened the local side can request the number of messages equal
to its buffer size, then send a new request for each message received.

Note the fact that this is only a single byte creates the limitation that you
can only request up to 255 additional messages at a time. This isn't ideal,
especially for byte streams where your chunk size could easily be small enough
to require sending multiple requests to communicate the local buffer size. I'm
very open to changes this to something bigger but want to keep everything to
one byte if possible. Want to test this more in practice before deciding.

```
MESSAGE_TYPE_CANCEL_SEND_STREAM
```

Instructs the remote end to destroy any knowledge of the stream, and cease
any processing or sending for said stream.
