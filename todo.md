* After calling end(), the onRequest() callback shouldn't be called again
* Consider allowing multiple event handlers, ie calls to onEnd shouldn't
  cancel previous callers who are expecting a notification.
* Consider adding Consumer.getDemand. This would allow connecting a downstream
  consumer to a conduit before the upstream producer has been connected. Might
  make it Conduit.getDemand instead
* Compare with yamux
* Need a way to allow data to be sent with the stream creation message.
* Consider renaming terminate (to cancel?), because it could be confused with
  the naturally end (termination) of a stream.
* Specify that pipe should return the consumer that was passed in, because if
  the consumer also implements Conduit it can be piped further.
