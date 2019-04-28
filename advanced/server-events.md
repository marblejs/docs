# Server events

The Node.js server after startup can emit a variety of different events that the app can listen to, eg. `upgrade`, `listening`, etc. As you know, streams are the main building block of Marble.js. **@marblejs/core** [createServer](../api-reference/core/createserver.md) function allows you to listen to emitted server events via exposed `event$` attribute, where you can hook your stream.

Similar to WebSocket and HTTP Effects, `HttpServerEffect` is used for dealing with stream of incoming server events, but in comparison to others, the Effect doesn't specify what will be the stream output.

```typescript
import { createServer, matchEvent, ServerEvent, HttpServerEffect, bindTo } from '@marblejs/core';

const listening$: HttpServerEffect = (event$, server, meta) =>
  event$.pipe(
    matchEvent(ServerEvent.listening),
    map(event => event.payload),
    tap(({ port, host }) => console.log(`Running @ http://${host}:${port}/`)),
  );

const server = createServer({
  // ...
  event$: (...args) => merge(
    listening$(...args),
    // ...
  ),
});

server.run();
```

As in the case of WebSockets, you can match incoming events using the same `matchEvent` operator.  `ServerEvent`  contains a full list of events that you can match to, preserving at the same time type correctness of matched events.

## Upgrading HTTP connections

> The [HTTP/1.1 protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP) provides a special mechanism that can be used to upgrade an already established connection to a different protocol, using the [`Upgrade`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade) header field.
>
> This mechanism is optional; it cannot be used to insist on a protocol change. Implementations can choose not to take advantage of an upgrade even if they support the new protocol, and in practice, this mechanism is used mostly to bootstrap a WebSockets connection.
>
> ---  
> source: [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)

```typescript
import { createServer, matchEvent, ServerEvent, HttpServerEffect, bindTo } from '@marblejs/core';
import { mapToServer } from '@marblejs/websockets';
import { WsServerToken } from './tokens';
import httpListener from './http.listener';
import webSocketListener from './ws.listener';

const upgrade$: HttpServerEffect = (event$, server, { ask }) =>
  event$.pipe(
    matchEvent(ServerEvent.upgrade),
    mapToServer({
      path: '/api/:version/ws',
      server: ask(WsServerToken),
    }),
  );

const server = createServer({
  port: 1337,
  httpListener,
  dependencies: [
    bindTo(WsServerToken)(webSocketListener().run),
  ],
  event$: (...args) => merge(
    upgrade$(...args),
    // ...
  ),
});

server.run();
```

Marble.js [Context](context.md) API plays well also with HTTP server Effects. You can upgrade running WebSocket server using dedicated [`mapToServer`](../api-reference/websockets/operator-maptoserver.md) operator. This kind of mechanism allows you to hook multiple WebSocket servers into already running HTTP server on different paths.

```typescript
// `createServer`
bindTo(WsServerToken_1)(webSocketListener_1().run),
bindTo(WsServerToken_2)(webSocketListener_2().run),

// upgrade$
mapToServer({
  path: '/ws_1',
  server: ask(WsServerToken_1),
}, {
  path: '/ws_2',
  server: ask(WsServerToken_2),
}),
```

