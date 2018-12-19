# Introduction

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
   and Go. This includes convenience functionality for setting up
   networked streams over WebSockets.
5. Extremely simple to understand and implement. If a reasonable amount of
   performance loss means a greatly simplified design, choose the simple
   option. A first-year undergrad CS student should be able to make a usefully
   complete implementation in an arbitrary programming language.


# Prior Work

omnistreams are heavily influenced by prior work, especially the excellent
[Reactive Streams](http://www.reactive-streams.org/), and
[ReactiveX](http://reactivex.io/). The primary differences are that omnistreams
have less functionality (no semantics for multiple subscribers, no built-in
operators, etc) and have a heavy focus on inter-language streaming. The most
important thing we stole from reactive streams is the pull-based backpressure
semantics (see also [pull
streams](https://github.com/pull-stream/pull-stream)).

There's also a lot of overlap with node streams. There several excellent
projects to bring node streams to the browser, and even between the browser and
node backend over websockets (see
[ws-streamify](https://github.com/baygeldin/ws-streamify)). You can think of
omnistreams as an attempt to generalize node streams to work in any
programming language, and between any two languages over a network.


# Interfaces

The following pseudocode outlines the primary interfaces.

## Core streams:

```
interface ConsumerStream<ElementType> {
    write: function(item: ElementType)
    end: function()
    onRequest: function(callback: function(numItems: uint32))

    cancel: function()
    onCancel: function(callback: function())
    onError: function(callback: function(error: string))
}

interface ProducerStream<ElementType> {
    request: function(numItems: uint32)
    onData: function(callback: function(data: ElementType))
    onEnd: function(callback: function())
    pipe: function(consumer: ConsumerStream<ElementType>)

    cancel: function()
    onCancel: function(callback: function())
    onError: function(callback: function(error: string))
}
```

## Channels (for streaming over a network, etc):

```
interface Channel<ElementType, MetadataType> {
    handleReceivedMessage: function(message: Array<uint8>)
    setSendHandler: function(message: Array<uint8>)
    createSendStream: function(metadata: MetadataType) -> ConsumerStream<ElementType>
    onReceiveStream: function(callback: function(stream: ProducerStream<ElementType>, metadata: MetadataType))
}
```

# Implementations (in order of completeness)

* [JavaScript](https://github.com/anderspitman/netstreams-js)
* [Go](https://github.com/anderspitman/omnistreams-go)
* [Rust](https://github.com/anderspitman/netstreams-rs)

# Additional Resources

* [Lossless backpressure in RxJS](https://itnext.io/lossless-backpressure-in-rxjs-b6de30a1b6d4)
* [Backpressure in Rx](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
* [pull streams](http://dominictarr.com/post/149248845122/pull-streams-pull-streams-are-a-very-simple)
