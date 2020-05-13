---
description: Creates messaging client reader for LOCAL transport layer (Event Bus)
---

# eventBus

### **Importing** <a id="importing"></a>

```typescript
import { eventBus } from '@marblejs/messaging';
```

### **Type declaration**

```haskell
eventBus :: EventBusConfig
  -> Reader<Context, Promise<TransportLayerConnection>>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `EventBusConfig` |

#### _**EventBusConfig**_

| _**parameter**_ | definition |
| :--- | :--- |
| _listener_ | &lt;optional&gt; `MessagingListener` |

{% hint style="warning" %}
Because of asynchronous nature of messaging client, all related readers have to be bound to server creators via eager binding 👉 [\#bindEagerlyTo](../core/bindeagerlyto.md)
{% endhint %}

To learn more about `eventBus` usage please visit:

{% page-ref page="../../../messaging/cqrs.md" %}

### Example

The event bus can be attached to server creator, like basic HTTP or any other microservice.

`@marblejs/messaging` module exposes already existing messaging client for `eventBus` transport layer:

```typescript
import { eventBus } from '@marblejs/messaging';
```

Additionally it exports tokens for both `EventBus` and `EventBusClient` instances.

```typescript
import { EventBusToken, EventBusClientToken } from '@marblejs/messaging';
```

```typescript
import { httpListener, createServer, bindEagerlyTo } from '@marblejs/core';
import { messagingListener, EventBusToken, EventBusClientToken, eventBusClient, eventBus } from '@marblejs/messaging';
​
const eventBusListener = messagingListener(...);
const listener = httpListener(...);
​
export const server = createServer({
  listener,
  dependencies: [
    bindEagerlyTo(EventBusClientToken)(eventBusClient),
    bindEagerlyTo(EventBusToken)(eventBus({ listener: eventBusListener })),
  ],
});
​
const main: IO<void> = async () =>
  await (await server)();
​
main();
```

