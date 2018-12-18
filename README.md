omnistreams aims to be a specification for a system to stream data around.
The project has the following core goals:

1. Language-agnostic interfaces and semantics for barebones read (producer) and
   write (consumer) streams.
2. Dead-simple wire protocol for multiplexing. This is for sending streams
   between networked machines, processes, threads, coroutines, etc. The only
   requirement is an underlying reliable, in-order, message passing system (ie
   WebSockets, WebRTC, ZeroMQ, reliable UDP, Go channels, etc)
3. Pull-based backpressure.
4. Reference implementations and base classes/interfaces in JavaScript, Rust,
   Go, and Python. This includes convenience functionality for setting up
   networked streams over WebSockets.
5. Extremely simple to understand and make new implementations. If a reasonable
   amount of performance loss means a greatly simplified design, choose the
   simple option. A first-year undergrad CS student should be able to make
   a usefully complete implementation in an arbitrary programming language.


omnistreams are heavily influenced by prior work, especially the excellent
[Reactive Streams](http://www.reactive-streams.org/). The primary differences
are that omnistreams have less functionality (no semantics for multiple
subscribers, for example) and have a heavy focus on inter-language operability.
The most important thing we stole from them is the pull-based backpressure
semantics (see also [pull streams](https://github.com/pull-stream/pull-stream).

There's also a lot of overlap with node streams. There several excellent
projects to bring node streams to the browser, and even between the browser and
node backend (see [ws-streamify](https://github.com/baygeldin/ws-streamify)).
You can think of omnistreams as an attempt to generalize node streams to work
in any language, and over a network.

The following pseudocode outlines the primary interfaces.

Core streams:

```
interface ConsumerStream<ItemType> {
    write: function(item: ItemType)
    end: function()
    onRequest: function(callback: function(numItems: uint32))

    cancel: function()
    onCancel: function(callback: function())
    onError: function(callback: function(error: string))
}

interface ProducerStream<ItemType> {
    request: function(numItems: uint32)
    onData: function(callback: function(data: ItemType))
    onEnd: function(callback: function())
    pipe: function(consumer: ConsumerStream<ItemType>)

    cancel: function()
    onCancel: function(callback: function())
    onError: function(callback: function(error: string))
}
```

Channels (for streaming over a network, etc):

```
interface Channel<ItemType> {
    handleReceivedMessage: function(message: Array<uint8>)
    setSendHandler: function(message: Array<uint8>)
    createSendStream: function(metadata: MetadataType) -> ConsumerStream<ItemType>
    onReceiveStream: function(callback: function(stream: ProducerStream<ItemType>, metadata: MetadataType))
}
```
