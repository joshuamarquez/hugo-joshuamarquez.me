+++
date = "2016-06-12T15:10:58-07:00"
draft = false
title = "How to use sails.io.js in Java"

+++

As we know [sails.io.js](https://github.com/balderdashy/sails.io.js) is the
Sails socket client that is bundled by default in new Sails apps. It is a
lightweight wrapper that sits on top of the Socket.IO client whose purpose is
to make sending and receiving messages from your Sails backend as simple as
possible.

Currently sails.io.js has only support for
[Browser](https://github.com/balderdashy/sails.io.js#for-the-browser) and
[Node.js](https://github.com/balderdashy/sails.io.js#for-nodejs). Here we will
learn how to make `.get()`, `.post()`, `.put()`, `.delete()` and `.request()`
methods from `sails.io.js` in **Java** with [socket.io-client-java](https://github.com/socketio/socket.io-client-java).
For `.off()` and `.on()` there is no need since `socket.io-client-java`
already does that.

Like surely you suspected `emit` method of `socket.io-client-java` is what we need
to make `sails.io.js` request. Lets take a look to
[_emitFrom](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L573) function from
[sails.io.js](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js) which is used to send requests.

```javascript
function _emitFrom(socket, requestCtx) {

  ...

  // Since callback is embedded in requestCtx,
  // retrieve it and delete the key before continuing.
  var cb = requestCtx.cb;
  delete requestCtx.cb;

  // Name of the appropriate socket.io listener on the server
  // ( === the request method or "verb", e.g. 'get', 'post', 'put', etc. )
  var sailsEndpoint = requestCtx.method;

  socket._raw.emit(sailsEndpoint, requestCtx, function serverResponded(responseCtx) {

    // Send back (emulatedHTTPBody, jsonWebSocketResponse)
    if (cb) {
      cb(responseCtx.body, new JWR(responseCtx));
    }
  });
}
```

As you can see, `sailsEndpoint` is the HTTP method to use as the socket event.
`requestCtx` is the body as the second argument of `emit`, and the third
parameter passed to `emit` is the  **Acknowledge callback**, a key parameter to
receive response from server. Once ``serverResponded`` is executed, the callback
extracted from `requestCtx` is called with response body and JSON web socket
response.

For example, if we do a `POST` with body request, `_emitFrom` will be executed
with the following `requestCtx`.

```javascript
{
  method, 'POST',
  headers: { 'x-my-custom-header': 'some string' },
  data: { email: 'joshua.marquezn@gmail.com' },
  url: '/user'
  cb: function(responseBody, JWR) {
    // ...
  }
}
```

As we mentioned before

* `method` is the event
* `headers`, `data` and `url` is the second argument of `.emit()`.
* `cb` is the Acknowledge callback of `.emit()`.

Now lets try to do something similar but in Java.

This could be be our [`_emitFrom`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L573) in Java.

```java
void emitFrom(Socket socket, SailsSocketRequest request) {
  // Name of the appropriate socket.io listener on the server
  // ( === the request method or "verb", e.g. 'get', 'post', 'put', etc. )
  String sailsEndpoint = request.getMethod();

  // Since Listener is embedded in request, retrieve it.
  final SailsSocketResponse.Listener listener = request.getListener();

  socket.emit(sailsEndpoint, request.toJSONObject(), new Ack() {
    @Override
    public void call(Object... args) {
      // Send back jsonWebSocketResponse
      if (listener != null) {
        listener.onResponse(new JWR((JSONObject) args[0]));
      }
    }
  });
}
```

You can find a full implementation of `sails.io.js` in Java called [`sails.io.java`](https://github.com/joshuamarquez/sails.io.java).
