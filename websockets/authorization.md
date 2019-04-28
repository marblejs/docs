# Connections handling

The WebSocket protocol doesn’t handle authorization or authentication. Practically, this means that a WebSocket opened from a page behind auth doesn’t “automatically” receive any sort of auth; you need to take steps to _also_ secure the WebSocket connection.

This can be done in a variety of ways, as WebSockets will pass through standard HTTP headers commonly used for authentication. This means you could use the same authentication mechanism you’re using on WebSocket connections as well.

The first possible solution assumes that the client will be authenticated during initial WebSocket connection. The `webSocketListener` config exposes a `connection$` stream of incoming connections that the developer can listen to and do some action on it. For example we can validate the incoming request headers and check for JWT token validity. In negative scenario we should throw an connection error with reason why it failed, and in positive scenario we should just pass the request through.

```typescript
import {
  webSocketListener,
  WebSocketConnectionError,
  WsConnectionEffect,
} from '@marblejs/websockets';

const connection$: WsConnectionEffect = req$ =>
  req$.pipe(
    mergeMap(req => iif(
      () => !isAuthorized(req)
      throwError(new WebSocketConnectionError('Unauthorized', 4000)),
      of(req),
    )),
  );
  
export const webSocketServer = webSocketListener({
  middlewares,
  effects,
  connection$,
});
```

