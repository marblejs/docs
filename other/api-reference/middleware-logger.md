---
description: HTTP request logger middleware for Marble.js
---

# middleware-logger

**Simple** middleware for request logging inside your console. It displays the outgoing request events using the following format:

```text
{HTTP_METHOD} {PATH} {HTTP_STATUS} {TIME}
```

```text
POST /api/v1/user 200 1ms
```

### Installation

```bash
$ yarn add @marblejs/middleware-logger
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { logger$ } from '@marblejs/middleware-logger';
```

### Type declaration

```text
logger$ :: LoggerOptions -> HttpMiddlewareEffect
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _options_ | &lt;optional&gt; `LoggerOptions` |

#### _**LoggerOptions**_

| _parameter_ | definition |
| :--- | :--- |
| _silent_ | &lt;optional&gt; `boolean` |
| _filter_ | &lt;optional&gt; `HttpRequest -> boolean` |

### Usage

1. Default behavior. Log every response to _process_._stdout_:

```typescript
import { httpListener } from '@marblejs/core';
import { logger$ } from '@marblejs/middleware-logger';

const middlewares = [
  logger$(),
  ...
];

export const listener = httpListener({
  middlewares,
  effects: [],
});
```

2. Customized logging behavior:

```typescript
import { httpListener } from '@marblejs/core';
import { logger$ } from '@marblejs/middleware-logger';
import { isTestEnv } from './util';

const middlewares = [
  logger$({
    silent: isTestEnv(),
    filter: req => req.res.status >= 400;
  }),
  ...
];

export const listener = httpListener({
  middlewares,
  effects: [],
});
```

* **silent** - When `true` the logging is turned off \(usually useful during testing\),
* **filter** - Filter outgoing responses or incoming requests based on given predicate. For example we can log only HTTP status codes above _400_.

