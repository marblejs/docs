# Server Events

The Node.js server after startup can emit a variety of different events that the app can listen to, eg. `upgrade`, `listening`, etc. As you know, streams are the main building block of Marble.js. **@marblejs/core** [createServer](../../other/api-reference/core/createserver.md) function allows you to listen to emitted server events via `event$` attribute, where you can hook your stream.

`HttpServerEffect` is used for dealing with stream of incoming server events, but in comparison to others, the interface doesn't specify what the output of the stream should be.

```typescript
import { createServer, matchEvent, ServerEvent, HttpServerEffect } from '@marblejs/core';
import { merge } from 'rxjs';
import { map, tap } from 'rxjs/operators';

const listening$: HttpServerEffect = event$ =>
  event$.pipe(
    matchEvent(ServerEvent.listening),
    map(event => event.payload),
    tap(({ port, host }) => console.log(`Running @ http://${host}:${port}/`)),
  );

const server = createServer({
  // ...
  event$: (...args) => merge(
    listening$(...args),
  ),
});
```

As in the case of WebSocket or messaging module, you can match incoming events using the same `matchEvent` operator.  `ServerEvent` contains a full list of events that you can match to, preserving at the same time type correctness of matched events.

## Upgrading HTTP connections

> The [HTTP/1.1 protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP) provides a special mechanism that can be used to upgrade an already established connection to a different protocol, using the [`Upgrade`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade) header field.
>
> This mechanism is optional; it cannot be used to insist on a protocol change. Implementations can choose not to take advantage of an upgrade even if they support the new protocol, and in practice, this mechanism is used mostly to bootstrap a WebSockets connection.
>
> ---  
> source: [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)

```typescript
import { createServer, matchEvent, ServerEvent, HttpServerEffect, bindEagerlyTo } from '@marblejs/core';
import { mapToServer } from '@marblejs/websockets';
import { merge } from 'rxjs';
import { WebSocketServerToken } from './tokens';
import { listener } from './http.listener';
import { webSocketServer } from './ws.listener';

const upgrade$: HttpServerEffect = (event$, ctx) =>
  event$.pipe(
    matchEvent(ServerEvent.upgrade),
    mapToServer({
      path: '/api/:version/ws',
      server: ctx.ask(WsServerToken),
    }),
  );

const server = createServer({
  port: 1337,
  httpListener,
  dependencies: [
    bindEageryTo(WebSocketServerToken)(async () =>
      await (await webSocketServer)()
    ),
  ],
  event$: (...args) => merge(
    upgrade$(...args),
  ),
});
```

You can upgrade running WebSocket server using dedicated [`mapToServer`](../../other/api-reference/websockets/operator-maptoserver.md) operator. This kind of mechanism allows you to hook multiple WebSocket servers into an already running HTTP server on different paths.

```typescript
// `createServer`
bindEageryTo(WsServerToken_1)(async () => await (await webSocketServer_1)()),
bindEageryTo(WsServerToken_2)(async () => await (await webSocketServer_2)()),

// upgrade$
mapToServer({
  path: '/ws_1',
  server: ctx.ask(WsServerToken_1),
}, {
  path: '/ws_2',
  server: ctx.ask(WsServerToken_2),
}),
```

