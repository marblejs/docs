---
description: >-
  AMQP is an open standard application layer protocol for message-oriented
  communication, i.e. implemented by RabbitMQ message-broker.
---

# AMQP \(RabbitMQ\)

> RabbitMQ is the most widely deployed open source message broker. With tens of thousands of users, it is one of the most popular open source message brokers. From [T-Mobile](https://www.youtube.com/watch?v=1qcTu2QUtrU) to [Runtastic](https://medium.com/@runtastic/messagebus-handling-dead-letters-in-rabbitmq-using-a-dead-letter-exchange-f070699b952b), it is used worldwide at small startups and large enterprises. RabbitMQ is lightweight and easy to deploy on premises and in the cloud. It supports multiple messaging protocols and can be deployed in distributed and federated configurations to meet high-scale, high-availability requirements.

## Installation

Before the usage remember to install required packages.

```bash
$ yarn add amqplib amqp-connection-manager
```

## Overview

{% tabs %}
{% tab title="microservice.ts" %}
```typescript
import { createMicroservice, Transport } from '@marblejs/messaging';

const microservice = createMicroservice({
  transport: Transport.AMQP
  options: {
    host: 'amqp://localhost:5672',
    queue: 'hello_queue',
    queueOptions: { durable: false },
  },
  // ...
});
```
{% endtab %}

{% tab title="client.ts" %}
```typescript
import { MessagingClient, createMicroservice, Transport } from '@marblejs/messaging';

const client = MessagingClient({
  transport: Transport.AMQP
  options: {
    host: 'amqp://localhost:5672',
    queue: 'hello_queue',
  },
  // ...
});
```
{% endtab %}
{% endtabs %}

The `options` property is specific to the chosen transport layer. **AMQP** exposes the following properties:

| Attribute | description |
| :--- | :--- |
| `host` | Host address of the RabbitMQ server |
| `queue` | Queue name which your microservice will listen to |
| `queueOptions` | Additional queue options \(see [`AssertQueue`](https://www.squaremobius.net/amqp.node/channel_api.html) specification\) |
| `prefetchCount` | Sets the prefetch count for the channel |
| `expectAck` | If `true`, manual acknowledgment mode is enabled |
| `timeout` | Timeout for RPC-like communication. Defaults to **120 seconds** |

### Acknowledgement

To make sure an event is never lost, RabbitMQ supports [message acknowledgements](https://www.rabbitmq.com/confirms.html). An acknowledgement is sent back by the consumer to tell RabbitMQ that a particular message has been received, processed and that RabbitMQ is free to delete it. If a consumer dies \(its channel is closed, connection is closed, or TCP connection is lost\) without sending an _ack_, RabbitMQ will understand that a message wasn't processed fully and will re-queue it.

To enable manual acknowledgment mode, set the `expectAck` property to `true`.

{% tabs %}
{% tab title="microservice.ts" %}
```typescript
import { createMicroservice, Transport } from '@marblejs/messaging';

const microservice = createMicroservice({
  transport: Transport.AMQP
  options: {
    host: 'amqp://localhost:5672',
    queue: 'hello_queue',
    expectAck: true, // 👈
  },
  // ...
});
```
{% endtab %}
{% endtabs %}

When manual acknowledgements are turned on, we must send a proper acknowledgement from the worker to signal that we are done with a task.

```typescript
import { MsgEffect, ackEvent, nackEvent } from '@marblejs/messaging';
import { matchEvent } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/function';
import { mapTo, tap, catchError } from 'rxjs/operators';

const foo$: MsgEffect = (event$, ctx) =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => pipe(
      // do some work...
      // ...and ack event
      tap(event => ackEvent(ctx)(event)()),
      mapTo({ type: 'FOO_RESPONSE' }),
      catchError(error => pipe(
        // nack event in case of an exception 
        defer(nackEvent(ctx)(event)),
        mergeMapTo(throwError(error)),
      ));
    )),
  );
```

**@marblejs/messaging** v3.3 introduces three handy functions for event acknowledgement: `ackEvent`, `nackEvent` and `nackAndResendEvent`. While the behavior of the first function is obvious, the second function will reject the event immediately, where the third one will try to resend the event again to the origin channel. You have to pass `EffectContext` first and the incoming event object to which acknowledgement will be made. Since all three functions are doing an asynchronous side effect, you have to call it explicitly.

```typescript
nackEvent(ctx)(event)() // Task<boolean> === () => Promise<boolean>
```

{% hint style="warning" %}
It is in the responsibility of developer to _nack_ messages. If the consumer won't _nack_ the message, it will hang up and wait for \(non-\)acknowledgement signal till the default transport layer timeout or the connection closing signal. After this time the event will be automatically rejected.
{% endhint %}

{% hint style="info" %}
If the microservice \(consumer\) is in acknowledgement mode \(`expectAck: true`\), all outgoing messages that target the same origin queue won't be emitted to prevent accidental consumer blocking. If you would like to emit another event, eg. as a confirmation response to the origin queue with acknowledgement mode enabled, you have to emit the event explicitly by connected messaging client.
{% endhint %}

