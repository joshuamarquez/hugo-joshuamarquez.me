+++
date = "2016-05-08T18:16:46-07:00"
draft = true
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
[sails.io.js](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js) which is used to send request.

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

As you can see, `sailsEndpoint` is the HTTP method to use. `requestCtx` is
the body, and the third parameter passed to `emit` is the **Acknowledge callback**
a key parameter to receive response from server. Once ``serverResponded`` is executed,
the callback extracted from `requestCtx` is called with response body and JSON web socket response.

Now we code something similar but in Java.

To simulate something similar to `requestCtx` JavaScript Object lets create
`WebSocketListener.java`, `WebSocketRequest.java` and `WebSocketResponse.java`.

### `WebSocketListener.java`

```java
public interface WebSocketListener {

  void onResponse(WebSocketResponse response);

  void onError(Error error);

}
```

## [`_emitFrom`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L573)

Next is to make something similar to [_emitFrom](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L573).

```java
private void emitFrom(WebSocketRequest request) {

    final WebSocketListener listener = request.getListener();
    JSONObject reqObj = null;

    try {
        reqObj = request.toJSONObject();
    } catch (JSONException e) {
        listener.onError(new Error(e));

        return;
    }

    if (reqObj != null) {
        mSocket.emit(request.getMethod(), reqObj, new Ack() {
            @Override
            public void call(Object... args) {
                listener.onResponse(new WSResponse(args[0]));
            }
        });
    } else {
        listener.onError(new Error("Error getting Socket Request!"));
    }

}
```

Once we have our `Java` classes ready is time to build the `sails.io.js` methods.

## [`request()`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L1200)


```java
public void request(String method, String url, JSONObject params, WebSocketListener listener) {
    emitFrom(new WebSocketRequest(method, url, params, listener));
}
```

## [`get()`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L1077)

```java
public void get(String url, JSONObject params, WebSocketListener listener) {
    emitFrom(new WebSocketRequest(WebSocketRequest.METHOD_GET, url, params, listener));
}
```

## [`post()`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L1105)

```java
public void post(String url, JSONObject params, WebSocketListener listener) {
    emitFrom(new WebSocketRequest(WebSocketRequest.METHOD_POST, url, params, listener));
}
```

## [`put()`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L1133)

```java
public void put(String url, JSONObject params, WebSocketListener listener) {
    emitFrom(new WebSocketRequest(WebSocketRequest.METHOD_PUT, url, params, listener));
}
```

## [`delete()`](https://github.com/balderdashy/sails.io.js/blob/master/sails.io.js#L1161)

```java
public void delete(String url, JSONObject params, WebSocketListener listener) {
    emitFrom(new WebSocketRequest(WebSocketRequest.METHOD_DELETE, url, params, listener));
}
```

Everything together will look like:

// TODO: methods in own WebSocket class.
