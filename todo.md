* After calling end(), the onRequest() callback shouldn't be called again
* Maybe createConsumer should be createStream after all, since it results in
  both a consumer and a producer (on the remote end) being created.
