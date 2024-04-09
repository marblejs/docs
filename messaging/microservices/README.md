---
description: >-
  Using a dedicated @marblejs/messaging module, Marble.js offers a robust and
  unified architecture for building event-based microservices.
---

# Microservices

{% hint style="warning" %}
This chapter doesn't intend to describe the idea that stands behind Microservices. If you would like to learn more about it, there are many good resources that explain the topic in a better way.
{% endhint %}

## Installation

```bash
$ yarn add @marblejs/messaging @marblejs/core rxjs fp-ts
```

## **Overview**

> **Microservice** architecture is an architectural style that structures an application as a collection of services that are: highly maintainable and testable, loosely coupled, independently deployable, organized around business capabilities, owned by a small team. The architecture enables the rapid, frequent and reliable delivery of large, complex applications.  
> [👉 https://microservices.io/](https://microservices.io/)

Microservices need a lightweight approach for communicating with one another. HTTP/1.1 is certainly a valid protocol but there are better options — especially when considering performance. While you might have used REST as your service communications layer in the past, more and more projects are moving to an event-driven architecture. When a service performs some piece of work that other services might be interested in, that service produces an _event_ — a record of the performed action. Other services consume those events so that they can perform any of their own tasks needed as a result of the event.

`@marblejs/messaging` defines the concept of transport layers, similar to NestJS. They are taking the responsibility of transporting messages between two parties. The messaging module abstracts the implementation details of each layer behind a generic Effect interface. For the sake of beginning, Marble.js 3.0 implements the [AMQP \(RabbitMQ\)](rabbitmq.md) and [Redis Pub/Sub](redis-pub-sub.md) transport layers.

{% hint style="info" %}
More transport layers are supposed to come in future minor releases, e.g. _NATS_, _MQTT_ or _GRPC_.
{% endhint %}

### Consumer

To instantiate a microservice, use the `createMicroservice()`.

{% tabs %}
{% tab title="consumer.ts" %}
```typescript
import { createMicroservice, messagingListener, Transport } from '@marblejs/messaging';
import { IO } from 'fp-ts/lib/IO';
import { hello$ } from './consumer.effect';

const listener = messagingListener({
  middlewares: [ ... ],
  effects: [ hello$ ],
});

const microservice = createMicroservice({
  transport: Transport.AMQP
  options: {
    host: 'amqp://localhost:5672',
    queue: 'hello_queue',
  },
  listener,
  dependencies: [ ... ],
});

const main: IO<void> = async () =>
  await (await microservice)();

main();
```
{% endtab %}
{% endtabs %}

The configuration object passed to factory function is very similar to other server factories, i.a. it allows to hook the _event$_, define dependencies and of course pass a listener. Besides that you have to obligatory define the transport layer and its options. The `options` object is always specific to the chosen transporter.

| Attribute | Description |
| :--- | :--- |
| `transport` | Specifies the transport layer \(e.g. `Transport.AMQP`\) |
| `options` | A transport-specific options object that determines transport layer behavior |

Microservice Effects can defined as any other `MsgEffect`. For more details about possible ways of processing events please visit messaging [Effects](../core-concepts/effects.md) chapter.

{% tabs %}
{% tab title="consumer.effect.ts" %}
```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect, reply } from '@marblejs/messaging';
import { of } from 'rxjs';

export const hello$: MsgEffect = (event$, ctx) =>
  event$.pipe(
    matchEvent('HELLO'),
    act(event =>
      of(reply(event)({
        type: event.type,
        payload: `Hello, ${event.payload}!`,
      })
    ),
  );
```
{% endtab %}
{% endtabs %}

### Publisher

A Publisher aka Client can exchange messages or publish events to a microservice.

{% tabs %}
{% tab title="publisher.client.ts" %}
```typescript
import { createContextToken } from '@marblejs/core';
import { messagingClient, MessagingClient, Transport } from '@marblejs/messaging';

export const ClientToken = createContextToken<MessagingClient>('MessagingClient');

export const client = messagingClient({
  transport: Transport.AMQP
  options: {
    host: 'amqp://localhost:5672',
    queue: 'hello_queue',
  },
});
```
{% endtab %}
{% endtabs %}

`messgingClient` factory function defines an interface similar to microservice factory: **transport** and **options**. Remember that you have to define your own context token for each messaging client you would like to bound to context. Messaging client exposes two main methods:

| Method | Description |
| :--- | :--- |
| `send` | Blocking \(**request-response**\), RPC-like messaging. |
| `emit` | Asynchronous messaging without response. |
| `close` | Closes active connection and teardowns subscribers. |

If you would like to communicate HTTP effects with your microservice just inject the bound client by specific token.

{% tabs %}
{% tab title="publisher.effect.ts" %}
```typescript
import { useContext } from '@marblejs/core';
import { r, HttpStatus } from '@marblejs/http';
import { mapTo, mergeMapTo } from 'rxjs/operators';
import { ClientToken } from './client';

export const getRoot$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => {
    const client = useContext(ClientToken)(ctx.ask);

    return req$.pipe(
      mergeMapTo(client.send({ type: 'HELLO', payload: 'John' })),
      mapTo({ status: HttpStatus.ACCEPTED }),
    );
  }),
);
```
{% endtab %}
{% endtabs %}

Remember that you have to bound the client to context before usage.

{% tabs %}
{% tab title="publisher.ts" %}
```typescript
import { bindEagerlyTo } from '@marblejs/core';
import { createServer, httpListener } from '@marblejs/http';
import { IO } from 'fp-ts/lib/IO';
import { client, ClientToken } from './client';
import { getRoot$ } from './effect';

const listener = httpListener({
  effects: [ getRoot$ ],
});

const server = createServer({
  listener,
  dependencies: [
    bindEagerlyTo(ClientToken)(client),
  ],
});

const main: IO<void> = async () =>
  await (await server)();

main();
```
{% endtab %}
{% endtabs %}

