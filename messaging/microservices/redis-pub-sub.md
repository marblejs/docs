---
description: >-
  Redis Pub/Sub implements the messaging system where the publishers sends the
  messages while the subscribers receive them.
---

# Redis Pub/Sub

> Aside from data storage, [Redis](https://redis.io/) can be used as a Publisher/Subscriber platform. In this pattern, publishers can issue messages to any number of subscribers on a channel. These messages are fire-and-forget, in that if a message is published and no subscribers exists, the message evaporates and cannot be recovered.

## Installation

Before the usage remember to install required packages.

```bash
$ yarn add redis
```

## Overview

{% tabs %}
{% tab title="microservice.ts" %}
```typescript
import { createMicroservice, Transport } from '@marblejs/messaging';

const microservice = createMicroservice({
  transport: Transport.REDIS
  options: {
    host: 'redis://127.0.0.1:6379',
    channel: 'hello_channel',
  },
  // ...
});
```
{% endtab %}

{% tab title="client.ts" %}
```typescript
import { createMicroservice, Transport } from '@marblejs/messaging';

const client = messagingClient({
  transport: Transport.REDIS
  options: {
    host: 'redis://127.0.0.1:6379',
    channel: 'hello_channel',
  },
  // ...
});
```
{% endtab %}
{% endtabs %}

The `options` property is specific to the chosen transport layer. **REDIS** exposes the following properties:

| Attribute | description |
| :--- | :--- |
| `host` | Host address of the Redis server |
| `channel` | Channel name which your microservice will listen to |
| `timeout` | Timeout for RPC-like communication. Defaults to **120 seconds** |
| `port` | Port of the Redis server |
| `password` | If set, client will run Redis auth command on connect. |

