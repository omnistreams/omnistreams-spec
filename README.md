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

# Spec

You can read the specification [here](spec.md)


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
[ws-streamify](https://github.com/baygeldin/ws-streamify), [websocket-stream](https://github.com/maxogden/websocket-stream)). In terms of
functionality offered (if you only need JavaScript support), omnistreams is very close to
[binaryjs](https://github.com/binaryjs/binaryjs).
Unfortunately, that project appears to be defunct. You can think of
omnistreams as an attempt to generalize node streams to work in any programming
language, and between any two languages over a network.


# Implementations (in order of completeness)

* [JavaScript](https://github.com/anderspitman/omnistreams-js)
* [Go](https://github.com/anderspitman/omnistreams-go)
* [Rust](https://github.com/anderspitman/netstreams-rs)


# Additional Resources

* [Lossless backpressure in RxJS](https://itnext.io/lossless-backpressure-in-rxjs-b6de30a1b6d4)
* [Backpressure in Rx](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
* [Node Streams](https://nodejs.org/api/stream.html)
* [Backpressuring in Streams (Node)](https://nodejs.org/en/docs/guides/backpressuring-in-streams/)
* [pull streams](http://dominictarr.com/post/149248845122/pull-streams-pull-streams-are-a-very-simple)
* [Haskell conduit package](https://github.com/snoyberg/conduit#readme)
* [Useful WHATWG Streams FAQ](https://github.com/whatwg/streams/blob/master/FAQ.md)
