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
import { createMicroservice, Transport } from '@marblejs/messaging';

const client = messagingClient({
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
import { MsgEffect } from '@marblejs/messaging';
import { matchEvent } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import { mapTo, tap, catchError } from 'rxjs/operators';

const foo$: MsgEffect = (event$, ctx) =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => pipe(
      // do some work ...
      tap(event => ctx.client.ackMessage(event.metadata?.raw)),
      mapTo({ type: 'FOO_RESPONSE' }),
      catchError(error => {
        ctx.client.nackMessage(event.metadata?.raw, false);
        return throwError(error);
      });
    )),
  );
```

`TransportLayerConnection` interface \(available through effect **ctx.client**\) defines two methods for managing events acknowledgement: `ackMessage()` and `nackMessage()`. In order to _ack_/_nack_ message you have to pass an initial event metadata raw object.

The second boolean parameter of `nackMessage` function defines if an event consumer should requeue failed event again or throw it away. By default it is set to `true` which means that every time the consumer will try to nack the incoming message it will be again requeued to origin channel \(queue\).

{% hint style="warning" %}
It is in the responsibility of developer to nack the errored message. If the consumer won't nack the message, it will hang and wait for non-acknowledgement infinitely till the connection won't be closed.
{% endhint %}

{% hint style="info" %}
If the microservice \(consumer\) is in acknowledgement mode \(`expectAck: true`\), all outgoing messages that target the same origin queue won't be emitted to prevent accidental consumer blocking. If you would like to emit another event, eg. as a confirmation response to the origin queue with acknowledgement mode enabled, you have to emit the event explicitly by connected messaging client.
{% endhint %}

