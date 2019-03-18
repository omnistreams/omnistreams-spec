* After calling end(), the onRequest() callback shouldn't be called again
* Consider allowing multiple event handlers, ie calls to onEnd shouldn't
  cancel previous callers who are expecting a notification.
* Consider adding Consumer.getDemand. This would allow connecting a downstream
  consumer to a conduit before the upstream producer has been connected. Might
  make it Conduit.getDemand instead
