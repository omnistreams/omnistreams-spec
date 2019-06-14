# Interfaces

The following pseudocode outlines the primary interfaces.

## Core interfaces:

```
interface Streamer {
    // Actions
    cancel: function()
    
    // Events
    onCancellation: function(callback: function())
    onError: function(callback: function(error: string))
}

interface Consumer {
    implements Streamer

    // Actions
    write: function(item: Array<uint8>)
    end: function()
    
    // Events
    onRequest: function(callback: function(numItems: uint8))
    onFinish: function(callback: function())
}

interface Producer {
    implements Streamer

    // Actions
    request: function(numItems: uint8)
    pipeInto: function(consumer: Consumer)
    pipeThrough: function(conduit: Conduit) -> Producer
    
    // Events
    onData: function(callback: function(data: Array<uint8>))
    onEnd: function(callback: function())
}

interface Conduit {
    implements Streamer
    implements Consumer
    implements Producer
}
```

## Multiplexers (for streaming over a network, etc):

```
interface Multiplexer {

    // Actions
    sendControlMessage: function(message: Array<uint8>)
    handleReceivedMessage: function(message: Array<uint8>)
    setSendHandler: function(callback: function(message: Array<uint8>))
    openConduit: function(metadata: Array<uint8>) -> Consumer
    
    // Events
    onControlMessage: function(callback: function(message: Array<uint8>))
    onConduit: function(callback: function(producer: Producer, metadata: Array<uint8>))
}
```


# Semantics

## Core interfaces:

```
interface Streamer {
```

Functionality common to both consumers and producers.


```
    cancel: function()
```

Cancels streaming early.


```
    onCancellation: function(callback: function())
```

Registers a callback to be notified if the streamer is canceled. Cancellation 
events are propagated upstream and downstream.


```
    onError: function(callback: function(error: string))
```

Registers a callback to be notified of any errors. Error events are propagated
upstream and downstream.

```
}
```

```
interface Consumer {
    implements Streamer
```

A sink for streaming data.

```
    write: function(item: Array<uint8>)
```

Writes bytes to be sent downstream. The calling application must never call
this method more times than have been requested via the callback passed to
onRequest, but it may call write less (ie if the upstream data source has ended
normally). The calling application is responsible for encoding messages to raw
byte sequences.

```
    end: function()
```

Called to indicate downstream that the data stream has ended normally.

```
    onRequest: function(callback: function(numItems: uint8))
```

Registers a callback to be called whenever more data has been requested from
downstream.

```
    onFinish: function(callback: function())
```

Registers a callback to be called when all data passed to write has been
successfully passed on (ie flushed).
```
}
```

```
interface Producer {
    implements Streamer
```

A source for streaming data.

```
    request: function(numItems: uint8)
```

Request more data from upstream. This method informs upstream of how much
data we are able to handle, therefore it can be used as a buffer. Typical
usage involves an initial call (or calls) to this method indicating how much
data is safe to send at one time (establishing the buffer size). Then each
time data is received this method is called again to request another message.

```
    onData: function(callback: function(data: Array<uint8>))
    onEnd: function(callback: function())
    pipeInto: function(consumer: Consumer)
    pipeThrough: function(conduit: Conduit) -> Producer
}

interface Conduit {
    implements Streamer
    implements Consumer
    implements Producer
}
```

## Multiplexers (for streaming over a network, etc):

```
interface Multiplexer {
    sendControlMessage: function(message: Array<uint8>)
    onControlMessage: function(callback: function(message: Array<uint8>))

    handleReceivedMessage: function(message: Array<uint8>)
    setSendHandler: function(message: Array<uint8>)
    openConduit: function(metadata: Array<uint8>) -> Consumer
    onConduit: function(callback: function(producer: Producer, metadata: Array<uint8>))
}
```
