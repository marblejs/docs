# Effects

Marble.js defines a common interface for many different kinds Effects. As you probably know, **@marblejs/core** defines a `HttpEffect` type for dealing with HTTP request streams. In case of **@marblejs/websockets**, the module defines a `WsEffect` which works within WebSocket protocol and deals with streams of Events.

## Hello, world!

Lets start with typical hello world example. The very basic implementation of WebSocket Effect can look like in the code snipped below.

{% code-tabs %}
{% code-tabs-item title="hello.ws-effect.ts" %}
```typescript
export const hello$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('HELLO'),
    mapTo({ type: 'HELLO', payload: 'Hello, world!' }),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Like any other Effect, it is just a function which returns a stream of outgoing events. As we previously mentioned, in case of WebSocket protocol, we have to deal with Events instead of requests.

The Effect above responds to _HELLO_ events with `Hello, world!` message. In case of default `WsEffect` interface, each incoming event has to be mapped to an outgoing event which is just an object with `type` and `payload` attributes.

## Calculator

Lets do some cool math! In the next example we will try to build a very basic calculator using only streams! For the example purpose we will only need two Effects: the first that will match _ADD_ events and the second that will match _SUM_ events.

{% code-tabs %}
{% code-tabs-item title="calculator.effect.ts" %}
```typescript
import { use, matchEvent } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { t, eventValidator$ } from '@marblejs/middleware-io';
import { buffer, map } from 'rxjs/operators';

export const sum$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('SUM')
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
{% endcode-tabs-item %}
{% endcode-tabs %}

As you can see, the WebSocket protocol is an ideal place for dealing with reactive streams, especially in Marble.js. In the example above we did a little bit of RxJS magic using `buffer` operator, which buffers the source Observable values until closing notifier emits \(in this case `sum$`\). Additionaly to be sure that incoming _ADD_ events are sent with payload of type number, we used [@marblejs/middleware-io](../api-reference/middleware-io.md) validator, which is able to infer payload type from defined schema.

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

Cool, right?! 😎 Don't forget to include the `add$` Effect in `webSocketListener`.

{% code-tabs %}
{% code-tabs-item title="webSocket.listener.ts" %}
```typescript
import { webSocketListener } from '@marblejs/websockets';
import { add$ } from './calculator.effect';

export const webSocketServer = webSocketListener({
  effects: [add$],
});

```
{% endcode-tabs-item %}
{% endcode-tabs %}

