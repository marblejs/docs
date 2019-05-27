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
$ npm i @marblejs/middleware-logger
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
| _stream_ | &lt;optional&gt; `WritableLike` |
| _filter_ | &lt;optional&gt; `(HttpResponse, HttpRequest) => boolean` |

### Usage

1. Default behaviour. Log every response to _process_._stdout_:

{% code-tabs %}
{% code-tabs-item title="logger.middleware.ts" %}
```typescript
import { logger$ } from '@marblejs/middleware-logger';

const middlewares = [
  logger$(),
  ...
];

export const app = httpListener({ middlewares, effects: [] });
```
{% endcode-tabs-item %}
{% endcode-tabs %}

2. Customized logging behaviour:

{% code-tabs %}
{% code-tabs-item title="logger.middleware.ts" %}
```typescript
import { logger$ } from '@marblejs/middleware-logger';
import { isTestEnv } from './util';

const middlewares = [
  logger$({
    silent: isTestEnv(),
    stream: createWriteStream(PATH, { flags: 'a' });;
    filter: (res, req) => res.status >= 400;
  }),
  ...
];

export const app = httpListener({ middlewares, effects: [] });
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **silent** - When `true` the logging is turned off \(usually useful during testing\),
* **stream** - Output stream for writing log messages, defaults to _process.stdout_. In the example above every response will be written to file pointed by provided `PATH` variable,
* **filter** - Filter outgoing responses or incoming requests based on given predicate. For example we can log only HTTP status codes above _400_.

