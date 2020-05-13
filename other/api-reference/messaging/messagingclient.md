---
description: Creates messaging client reader for given transport layer
---

# messagingClient

### **Importing** <a id="importing"></a>

```typescript
import { messagingClient } from '@marblejs/messaging';
```

### **Type declaration**

```haskell
messagingClient :: MessagingClientConfig
    -> Reader<Context, Promise<MessagingClient>>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `MessagingClientConfig` |

#### _**MessagingClientConfig**_

| _**parameter**_ | definition |
| :--- | :--- |
| _msgTransformer_ | &lt;optional&gt; `TransportMessageTransformer` |
| _transport_ | `Transport` \(see: [\#createMicroservice](createmicroservice.md#createmicroserviceconfig)\) |
| options | `StrategyOptions` \(see: [\#createMicroservice](createmicroservice.md#createmicroserviceconfig)\) |

{% hint style="warning" %}
Because of asynchronous nature of messaging client, all created readers have to be bound to server creators via eager binding 👉 [\#bindEagerlyTo](../core/bindeagerlyto.md)
{% endhint %}

To learn more about `messagingClient` output interface please visit:

{% page-ref page="../../../messaging/microservices/" %}

### Example \(AMQP\):

```typescript
import { bindEagerlyTo, createContextToken } from '@marblejs/core';
import { messagingClient, Transport } from '@marblejs/messaging';

const AmqpClientToken = createContextToken<MessagingClient>('AmqpClient');

const AmqpClient = messagingClient({
  transport: Transport.AMQP,
  options: {
    host: 'amqp://localhost:5672',
    queue: 'some_queue_name',
    queueOptions: { durable: false },
    timeout: 360 * 1000,
  },
});

...

const dependencies = [
  bindEagerlyTo(AmqpClientToken)(AmqpClient),
];

```

### Example \(REDIS\):

```typescript
import { bindEagerlyTo, createContextToken } from '@marblejs/core';
import { messagingClient, Transport } from '@marblejs/messaging';

const RedisClientToken = createContextToken<MessagingClient>('RedisClient');

const RedisClient = messagingClient({
  transport: Transport.REDIS,
  options: {
    host: 'redis://127.0.0.1:6379',
    channel: 'some_channel_name',
    timeout: 360 * 1000,
  },
});

...

const dependencies = [
  bindEagerlyTo(RedisClientToken)(RedisClient),
];

```

