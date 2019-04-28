# webSocketListener

### **Importing**

```typescript
import { webSocketListener } from '@marblejs/websockets';
```

### **Type declaration**

```typescript
webSocketListener :: WebSocketListenerConfig -> WebSocket.ServerOptions -> ContextReader
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `WebSocketListenerconfig` |

#### _**WebSocketListenerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| _effects_ | &lt;optional&gt; `Array<WsEffect>` |
| _middlewares_ | &lt;optional&gt; `Array<WsMiddlewareEffect>` |
| _error$_ | &lt;optional&gt; `WsErrorEffect` |
| connection$ | &lt;optional&gt; `WsConnectionEffect` |
| output$ | &lt;optional&gt; `WsOutputEffect` |
| eventTransformer | &lt;optional&gt; `EventTransformer` |

### **Example**

{% code-tabs %}
{% code-tabs-item title="websocket.listener.ts" %}
```typescript
import { webSocketListener } from '@marblejs/websockets';
import { example$ } from './example.effect';
import { logger$ } from './logger.middleware';

export default webSocketListener({
  middlewares: [logger$],
  effects: [add$],
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

