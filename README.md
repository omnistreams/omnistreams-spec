# Introduction

omnistreams is a specification defining a data streaming system.
The project has the following core goals:

1. Language-agnostic interfaces and semantics for barebones readable (producer) and
   writable (consumer) streams, including piping, cancellation, and operation (ie map, filter, etc) support.
2. Dead-simple wire protocol with multiplexing. This is for sending streams
   between networked machines, processes, threads, coroutines, etc. The only
   requirement is an underlying reliable, in-order, message passing system (ie
   WebSockets, WebRTC, ZeroMQ, reliable UDP, Go channels, etc)
3. Pull-based (ie client-controlled) backpressure.
4. Reference implementations and base classes/interfaces in JavaScript, Rust,
   and Go. This includes convenience functionality for setting up
   networked streams over WebSockets, ie a Multiplexer implementation for
   WebSockets.
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

There's also a lot of overlap with node streams. There are several excellent
projects to bring node streams to the browser, and even between the browser and
node backend over websockets (see
[ws-streamify](https://github.com/baygeldin/ws-streamify), [websocket-stream](https://github.com/maxogden/websocket-stream)). In terms of
functionality offered (if you only need JavaScript support), omnistreams is very close to
[binaryjs](https://github.com/binaryjs/binaryjs).
Unfortunately, that project appears to be defunct. You can think of
omnistreams as an attempt to generalize node streams to work in any programming
language, and between any two languages over a network.

I discovered [rsocket](http://rsocket.io/) some time after completing a working JavaScript implementation of omnistreams. The goals of the two projects appear to be **very** similar, with rsocket being far more complete and mature. If I had found it when I went looking for a language-agnostic backpressured WebSocket stream multiplexer, I would have spent some time trying to make it work before implementing my own solution. As it stands, the biggest advantage of omnistreams that I see is that it has a much simpler protocol, mostly because it offers far fewer features. This makes it faster to implement for new languages and easier for developers to pick up and use. Knowing that rsocket exists gives me confidence in choosing to keep omnistreams as simple as possible. Also, omnistreams was built from the ground up to support raw byte streams, ie transferring files, in addition
to streaming things like JSON or Protobuf objects. Everything is just bytes and the application is responsible for encoding and
decoding. I'm sure other systems support byte streams, but it doesn't seem to be a focus. omnistreams is very agnostic to the datatype. 


# Implementations (in order of completeness)

* [JavaScript](https://github.com/omnistreams/omnistreams-js)
* [Rust](https://github.com/omnistreams/omnistreams-rs)
* [Go](https://github.com/omnistreams/omnistreams-go)


# Additional Resources

* [Excellent rsocket FAQ](http://rsocket.io/docs/FAQ)
* [Excellent rsocket Motivations doc](http://rsocket.io/docs/Motivations)
* [Lossless backpressure in RxJS](https://itnext.io/lossless-backpressure-in-rxjs-b6de30a1b6d4)
* [Backpressure in Rx](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
* [Node Streams](https://nodejs.org/api/stream.html)
* [Backpressuring in Streams (Node)](https://nodejs.org/en/docs/guides/backpressuring-in-streams/)
* [pull streams](http://dominictarr.com/post/149248845122/pull-streams-pull-streams-are-a-very-simple)
* [Haskell conduit package](https://github.com/snoyberg/conduit#readme)
* [Informative WHATWG Streams FAQ](https://github.com/whatwg/streams/blob/master/FAQ.md)
* [Informative Firefox Streams Bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1128959)
