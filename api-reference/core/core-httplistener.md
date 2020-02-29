---
description: >-
  Starting point of every Marble.js application. It includes definitions of all
  middlewares and API effects.
---

# httpListener

### **Importing**

```typescript
import { httpListener } from '@marblejs/core';
```

### **Type declaration**

```typescript
httpListener :: HttpListenerConfig -> (IncomingMessage, OutgoingMessage) -> void;
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `HttpListenerConfig` |
| _req_  | Node.js `Http.IncomingMessage` |
| res | Node.js `Http.OutgoingMessage` |

#### _**HttpListenerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| _effects_ | `Array<RouteEffect | RouteEffectGroup>` |
| _middlewares_ | &lt;optional&gt; `Array<HttpMiddlewareEffect>` |
| _error$_ | &lt;optional&gt; `HttpErrorEffect` |

### Returns

Besides the default http server handler, the `httpListener` returns also an configuration object.

| parameter | definitoin |
| :--- | :--- |
| _routing_ | `Array<RoutingItem>` |
| injector | `StaticInjector` |

### **Example**

{% code title="http.listener" %}
```typescript
import { httpListener } from '@marblejs/core';
import { bodyParser$ } from '@marblejs/middleware-body';
import { logger$ } from '@marblejs/middleware-logger';
import { api$ } from './api';

const middlewares = [
  logger$(),
  bodyParser$(),
];

const effects = [
  api$,
];

export default httpListener({ middlewares, effects });
```
{% endcode %}

