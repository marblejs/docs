---
description: >-
  Starting point of every Marble.js application. It includes definitions of all
  middlewares and API effects.
---

# core: httpListener

### **Importing**

```typescript
import { httpListener } from '@marblejs/core';
```

### **Type declaration**

```typescript
httpListener :: HttpListenerConfig -> (IncomingMessage, OutgoingMessage) -> void
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
| _middlewares_ | &lt;optional&gt; `Array<Middleware>` |
| _errorEffect_ | &lt;optional&gt; `ErrorEffect` |

### Returns

`void`

### **Example**

{% code title="app.ts" %}
```typescript
import { httpListener } from '@marblejs/core';
import { bodyParser$ } from '@marblejs/middleware-body';
import { logger$ } from '@marblejs/middleware-logger';
import { api$ } from './api';

const middlewares = [
  logger$,
  bodyParser$,
];

const effects = [
  api$,
];

export const app = httpListener({ middlewares, effects });
```
{% endcode %}

