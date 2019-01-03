* After calling end(), the onRequest() callback shouldn't be called again
* Consider allowing multiple event handlers, ie calls to onEnd shouldn't
  cancel previous callers who are expecting a notification.
