# Middlewares

## Custom middleware

Like any other Effect, Marble.js defines a similar way for defining middlewares. In case of WebSocket protocol it is a function that transforms _stream of incoming events_ 👉 _stream of outgoing events_.

Lets create a simple logger middleware like we previously did in case of HTTP.

{% code-tabs %}
{% code-tabs-item title="logger.ws-middleware.ts" %}
```typescript
import { Event } from '@marblejs/core';
import { WsMiddlewareEffect } from '@marblejs/websockets';

export const logger$: WsMiddlewareEffect = event$ =>
  event$.pipe(
    tap(e => console.log(`type: ${e.type}, payload: ${e.payload}`)),
  );
  
// or an equivalent

export const logger$ = (event$: Observable<Event>): Observable<Event> =>
  event$.pipe(
    tap(e => console.log(`type: ${e.type}, payload: ${e.payload}`)),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Similar to any other middleware or effect, we have to register it inside the listener.

{% code-tabs %}
{% code-tabs-item title="webSocket.listener.ts" %}
```typescript
import { webSocketListener } from '@marblejs/websockets';
import { logger$ } from './logger.ws-middleware';

export const webSocketServer = webSocketListener({
  middlewares: [logger$],
  // ...
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Middlewares composition

Similar to HTTP routes, you can compose middlewares in two ways:

* globally \(inside `webSocketListener` configuration object\),
* by composing it directly inside _Effect_ event pipeline.

The [use](../api-reference/core/operator-use.md) operator availabe in [@marblejs/core](../api-reference/core/) package allows you to compose multiple types of middlewares - not only HTTP-related. Lets examine how can we compose the [@marblejs/middleware-io](../api-reference/middleware-io.md) middleware in `event$` pipeline.

```typescript
import { use } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { eventValidator$, t } from '@marblejs/middleware-io';

const User = t.type({
  name: t.string,
  age: t.number,
});

export const createUser$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('CREATE_USER'),
    use(eventValidator$(User)),
    // ...   event.payload: { name: string, age: number }
  );
```



