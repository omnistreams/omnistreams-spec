# Terminology

Terms such as stream, channel, publisher, subscriber, etc sometimes have
different meanings in different contexts. This spec aims to be somewhat
pedantic, using terms primarily based on their English meaning to represent
the underlying concepts. While this will hopefully make things simpler for
beginners, it also means sometimes breaking with common uses of the terms.

* **Stream** - A stream in omnistreams represents the data itself. So each
  item/chunk received from Producer.onData events is part of the stream. This
  especially
  breaks with systems like node, where the stream is the object that produces
  or consumes the data. omnistreams are also always unidirectional. If you need data
  flowing in both directions, open another stream.
* **Producer** - a Producer emits a stream of data. This would be called a
  ReadStream in node. I found the terms Read/Write stream confusing since it
  could mean you read data from the stream, or the stream reads your data and
  passes it on.
* **Consumer** - A Consumer is a sink for a data stream. Analogous to a WriteStream
  in node.
* **Conduit** - A Conduit acts as both a Consumer and a Producer, with a single
  unidirectional stream being fed to the Consumer and output by the Producer.
  Conduits have two primary uses. The first is for modifying a data stream in
  some way, in which case it is a transform Conduit. This is how you would
  implement the concept of operators from systems like Rx. The other us is for
  bridging
  concurrency barriers, ie passing a stream over a network or between threads.
  Note that the term Channel would probably usually be used for this purpose.
  However, that term is heavily used for several different meanings, so we
  chose to use a different term to avoid confusion. We believe this usage is
  essentially the same as the Haskell conduit package (see links below).
* **Multiplexer** - Multiplexers provide a simple interface for creating and
  managing multiple Conduits over a single concurrency medium. This is useful
  for sharing a single network socket or other communication primitive where
  it's undesirable to open a new underlying resource for every stream.


The spec has two primary sections:

* [Language-Independent Interfaces](interfaces.md)
* [Wire Protocol](wire-protocol.md)

