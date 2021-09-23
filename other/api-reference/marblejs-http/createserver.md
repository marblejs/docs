---
description: Creates HTTP server
---

# createServer

### I**mporting**

```typescript
import { createServer } from '@marblejs/http';
```

### **Type declaration**

```haskell
createServer ::  CreateServerConfig -> () -> Promise<ServerIO<HttpServer>>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `CreateServerConfig` |

#### _**CreateServerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| _listener_ | `HttpListener` |
| _port_ | &lt;optional&gt; `number` |
| _hostname_ | &lt;optional&gt; `string` |
| _event$_ | &lt;optional&gt; `HttpServerEffect` |
| _options_ | &lt;optional&gt; `ServerOptions` |
| _dependencies_ | &lt;optional&gt; `Array<BoundDependency<any>>` |

### Returns

`ServerIO<HttpServer>is` 

### **Example**

```typescript
import { bindTo } from '@marblejs/core';
import { createServer } from '@marblejs/http';
import { IO } from 'fp-ts/lib/IO';
import { listener } from './http.listener';

const httpsOptions: https.ServerOptions = {
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem'),
};

const server = createServer({
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

const main: IO<void> = async () =>
  await (await server)();

main();
```

