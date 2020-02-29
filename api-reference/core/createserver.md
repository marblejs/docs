---
description: Marble.j abstraction over Node.js server creation with additional features.
---

# createServer

### I**mporting**

```typescript
import { createServer } from '@marblejs/core';
```

### **Type declaration**

```text
createServer ::  CreateServerConfig -> Server
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `CreateServerConfig` |

#### _**CreateServerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| _httpListener_ | `HttpListener` |
| _port_ | &lt;optional&gt; `number` |
| _hostname_ | &lt;optional&gt; `string` |
| _event$_ | &lt;optional&gt; `HttpServerEffect` |
| _options_ | &lt;optional&gt; `ServerOptions` |
| _dependencies_ | &lt;optional&gt; `Array<BoundDependency<any>>` |

### Returns

| parameter | definitoin |
| :--- | :--- |
| _run_ | `boolean? -> https.Server | http.Server` |
| _info_ | `ServerInfo` |
| _server_ | `https.Server | http.Server` |

_**ServerInfo**_

| parameter | definitoin |
| :--- | :--- |
| _routing_ | `Array<RoutingItem>` |

### **Example**

{% code title="index.ts" %}
```typescript
import httpListener from './http.listener';
import { createServer, bindTo } from '@marblejs/core';

const httpsOptions: https.ServerOptions = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem'),
};

export const server = createServer({
  port: 1337,
  hostname: '127.0.0.1',
  httpListener,
  dependencies: [
    bindTo(fooToken)(foo),
  ],
  event$: (...args) => merge(
    listening$(...args),
    upgrade$(...args),
  ),
  options: { httpsOptions },
});

server.run(
  process.env.NODE_ENV !== 'test'
);
```
{% endcode %}

