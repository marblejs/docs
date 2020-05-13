---
description: Creates and bootstraps microservice for given transport layer
---

# createMicroservice

### **Importing** <a id="importing"></a>

```typescript
import { createMicroservice } from '@marblejs/messaging';
```

### **Type declaration**

```haskell
createMicroservice :: CreateMicroserviceConfig
    -> Promise<ServerIO<TransportLayerConnection>>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `CreateMicroserviceConfig` |

#### _\*\*\*\*_

#### _**CreateMicroserviceConfig**_

| _**parameter**_ | definition |
| :--- | :--- |
| _listener_ | `MessagingListener` |
| _event$_ | &lt;optional&gt; `HttpServerEffect` |
| _dependencies_ | &lt;optional&gt; `Array<BoundDependency<any>>` |
| transport | `Transport` |
| options | `StrategyOptions` |

#### _**StrategyOptions \(**`Transport.AMQP`**\)**_

| _parameter_ | definition |
| :--- | :--- |
| _host_ | `string` |
| _queue_ | `string` |
| _queueOptions_ | &lt;optional&gt; `Options.AssertQueue` \(see: [amqplib](https://www.squaremobius.net/amqp.node/channel_api.html#channel_assertQueue)\) |
| _prefetchCount_ | &lt;optional&gt; `number` \(defaults to **1**\) |
| _expectAck_ | &lt;optional&gt; boolean |
| _timeout_ | &lt;optional&gt; `number` in ms \(defaults to **120s**\) |

#### _**StrategyOptions \(**`Transport.REDIS`**\)**_

| _parameter_ | definition |
| :--- | :--- |
| _host_ | `string` |
| _channel_ | `string` |
| port | &lt;optional&gt; `number` |
| _password_ | &lt;optional&gt; `string` |
| _timeout_ | &lt;optional&gt; `number` in ms \(defaults to **120s**\) |

### Example \(AMQP\):

```typescript
import { IO } from 'fp-ts/lib/IO';
import { createMicroservice, messagingListener, Transport } from '@marblejs/messaging';

const amqpMicroservice = createMicroservice({
  transport: Transport.AMQP,
  options: {
    host: 'amqp://localhost:5672',
    queue: 'some_queue_name',
    queueOptions: { durable: true },
    timeout: 360 * 1000,
  },
  listener: messagingListener(...),
  dependencies: [...],
});

const main: IO<void> = async () =>
  (await amqpMicroservice)()

main();
```

### Example \(REDIS\):

```typescript
import { IO } from 'fp-ts/lib/IO';
import { createMicroservice, messagingListener, Transport } from '@marblejs/messaging';

const redisMicroservice = createMicroservice({
  transport: Transport.REDIS,
  options: {
    host: 'redis://127.0.0.1:6379',
    channel: 'some_channelname',
    timeout: 360 * 1000,
  },
  listener: messagingListener(...),
  dependencies: [...],
});

const main: IO<void> = async () =>
  (await redisMicroservice)()

main();
```

