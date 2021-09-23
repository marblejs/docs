---
description: >-
  Starting point of every Marble.js application. It includes definitions of all
  middlewares and API effects.
---

# httpListener

### **Importing**

```typescript
import { httpListener } from '@marblejs/http';
```

### **Type declaration**

```typescript
httpListener :: HttpListenerConfig -> Reader<Context, HttpListener>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `HttpListenerConfig` |

#### _**HttpListenerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| _effects_ | &lt;optional&gt; `Array<RouteEffect | RouteEffectGroup>` |
| _middlewares_ | &lt;optional&gt; `Array<HttpMiddlewareEffect>` |
| _error$_ | &lt;optional&gt; `HttpErrorEffect` |
| _output$_ | &lt;optional&gt; `HttpOutputEffect` |

### Returns

`Reader<Context, HttpListener>`

### **Example**

{% code title="http.listener" %}
```typescript
import { httpListener } from '@marblejs/http';
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

export const listener = httpListener({ middlewares, effects });
```
{% endcode %}

