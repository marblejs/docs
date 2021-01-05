---
description: >-
  @marblejs/websockets module implements the RFC 6455 WebSocket protocol,
  providing full-duplex communication channels over a single TCP connection.
---

# WebSockets

## Installation

```bash
$ yarn add @marblejs/websockets @marblejs/core rxjs fp-ts
```

## Bootstrapping

Like [_httpListener_](../other/api-reference/core/core-httplistener.md) \_\_the WebSocket module defines a similar way of bootstrapping the app.

{% tabs %}
{% tab title="webSocket.listener.ts" %}
```typescript
import { webSocketListener } from '@marblejs/websockets';

const effects = [
  effect1$,
  effect2$,
  // ...
];

const middlewares = [
  middleware1$,
  middleware2$,
  // ...
];

export const listener = webSocketListener({ effects, middlewares });
```
{% endtab %}
{% endtabs %}

To create WebSocket app instance, we can use `createWebSocketServer`, which is a wrapper around `ws` server creator. When created, it won't automatically start listening to given port and hostname until you call its awaited instance.

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { createWebSocketServer } from '@marblejs/websockets';
import { IO } from 'fp-ts/lib/IO';
import { listener } from './webSocket.listener';

const webSocketServer = createWebSocketServer({
  options: {
    port: 1337,
    host: '127.0.0.1',
  }, 
  listener,
});

const main: IO<void> = async () =>
  await (await webSocketServer)();

main();
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you are curious about other ways of server bootstrapping, reach out the HTTP [Server events](../http/advanced/server-events.md) chapter.
{% endhint %}

## Effects

Marble.js defines a common interface for many different kinds Effects. In case of **@marblejs/websockets**, the module defines a `WsEffect` which works within WebSocket protocol and deals with streams of Events. The very basic implementation of WebSocket Effect can look like in the code snipped below.

{% tabs %}
{% tab title="hello.effect.ts" %}
```typescript
import { matchEvent } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { mapTo } from 'rxjs/operators';

export const hello$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('HELLO'),
    mapTo({ type: 'HELLO', payload: 'Hello, world!' }),
  );
```
{% endtab %}
{% endtabs %}

Like any other Effect, it is just a function which returns a stream of outgoing events. The Effect above responds to _HELLO_ events with `Hello, world!` message. In case of default `WsEffect` interface, each incoming event has to be mapped to an outgoing [event](core-concepts/events.md) which is just an object with **type** and **payload** attributes.

Lets do some cool math! In the next example we will try to build a very basic calculator using only streams! For the example purpose we will only need two Effects: the first that will match _ADD_ events and the second that will match _SUM_ events.

{% tabs %}
{% tab title="calculator.effect.ts" %}
```typescript
import { use, matchEvent } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { t, eventValidator$ } from '@marblejs/middleware-io';
import { buffer, map } from 'rxjs/operators';

export const sum$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('SUM'),
  );

export const add$: WsEffect = (event$, ...args) =>
  event$.pipe(
    matchEvent('ADD'),
    use(eventValidator$(t.number)),
    buffer(sum$(event$, ...args)),
    map(events => events.reduce((a, e) => e.payload + a, 0)),
    map(payload => ({ type: 'SUM_RESULT', payload })),
  );
```
{% endtab %}
{% endtabs %}

In the example above we did a little bit of RxJS magic using `buffer` operator, which buffers the source Observable values until closing notifier emits \(in this case `sum$`\). Additionally, to be sure that incoming _ADD_ events are sent with payload of type number, we used [@marblejs/middleware-io](../other/api-reference/middleware-io.md) validator, which is able to infer payload type from defined schema.

Lets examine in steps how the server will respond to given equation: $$7 + 3 + 1$$

```javascript
// #1 sent to server
{
  "type": "ADD",
  "payload": 7
}

// #2 sent to server
{
  "type": "ADD",
  "payload": 3
}

// #3 sent to server
{
  "type": "ADD",
  "payload": 1
}

// #4 sent to server
{
  "type": "SUM",
}

// #5 response from server
{
  "type": "SUM_RESULT",
  "payload": 11
}
```

