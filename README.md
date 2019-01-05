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


# Terminology

Terms such as stream, channel, publisher, subscriber, etc sometimes have
different meanings in different contexts. This spec aims to be somewhat
pedantic, using terms primarily based on their English meaning to represent
the underlying concepts. While this will hopefully make things simpler for
beginners, it also means sometimes breaking with common uses of the terms.

* Stream - A stream in omnistreams represents the data itself. This especially
  breaks with systems like node, where the stream is the object that provides
  the data. omnistreams are also always unidirectional. If you need data
  flowing in both directions, set up another stream.
* Producer - a Producer emits a stream of data. This would be called a
  ReadStream in node. I found the terms Read/Write stream confusing since it
  could mean you read data from the stream, or the stream reads your data and
  passes it on.
* Consumer - A Consumer is a sink for a data stream. Analogous to a WriteStream
  in node.
* Conduit - A Conduit acts as both a Consumer and a Producer, with a single
  unidirectional stream being fed to the Consumer and output by the Producer.
  Conduits have two primary uses. The first is for modifying a data stream in
  some way, in which case it is a transform Conduit. The other is for bridging
  concurrency barriers, ie passing a stream over a network or between threads.
  Note that the term Channel would probably usually be used for this purpose.
  However, that term is heavily used for several different meanings, so we
  chose to use a different term to avoid confusion. I believe this usage is
  essentially the same as the Haskell conduit package (see links below).
* Multiplexer - Multiplexers provide a simple interface for creating and
  managing multiple Conduits over a single concurrency medium. This is useful
  for sharing a single network socket or other communication primitive where
  it's undesirable to open a new underlying resource for every stream.


# Interfaces

The following pseudocode outlines the primary interfaces.

## Core interfaces:

```
interface Streamer {
    terminate: function()
    onTermination: function(callback: function())
    onError: function(callback: function(error: string))
}

interface Consumer {
    implements Streamer

    write: function(item: Array<uint8>)
    end: function()
    onRequest: function(callback: function(numItems: uint8))
    onFinish: function(callback: function())
}

interface Producer {
    implements Streamer

    request: function(numItems: uint8)
    onData: function(callback: function(data: Array<uint8>))
    onEnd: function(callback: function())
    pipe: function(consumer: Consumer)
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
    createConduit: function(metadata: Array<uint8>) -> Consumer
    onConduit: function(callback: function(producer: Producer, metadata: Array<uint8>))
}
```


# Implementations (in order of completeness)

* [JavaScript](https://github.com/anderspitman/omnistreams-js)
* [Go](https://github.com/anderspitman/omnistreams-go)
* [Rust](https://github.com/anderspitman/netstreams-rs)


# Additional Resources

* [Lossless backpressure in RxJS](https://itnext.io/lossless-backpressure-in-rxjs-b6de30a1b6d4)
* [Backpressure in Rx](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
* [pull streams](http://dominictarr.com/post/149248845122/pull-streams-pull-streams-are-a-very-simple)
* [Haskell conduit package](https://github.com/snoyberg/conduit#readme)
