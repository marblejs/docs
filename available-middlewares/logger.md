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
import { loggerWithOpts$ } from '@marblejs/middleware-logger';
```

{% hint style="warning" %}
From version **v1.2** the `logger$` entry point is marked as deprecated. Use `loggerWithOpts$` instead. From the version **v2.x** the old entry point will be swapped with newer implementation.
{% endhint %}

### Type declaration

```text
loggerWithOpts$ :: (LoggerOptions) -> Middleware
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _options_ | &lt;optional&gt; `LoggerOptions` |

#### _**LoggerOptions**_

| _parameter_ | definition |
| :--- | :--- |
| _silent_ | &lt;optional&gt; `boolean` |
| _stream_ | &lt;optional&gt; `WriteStream` |
| _filter_ | &lt;optional&gt; `(HttpResponse) => boolean` |

### Usage

1. Default behaviour. Log every response to _process_._stdout_ \(_console.log_\):

{% code title="logger.middleware.ts" %}
```typescript
import { loggerWithOpts$ } from '@marblejs/middleware-logger';

export const logger$ = loggerWithOpts$();
```
{% endcode %}

2. Customized logging behaviour:

{% code title="logger.middleware.ts" %}
```typescript
import { loggerWithOpts$ } from '@marblejs/middleware-logger';
import { isTestEnv } from './util';

export const logger$ = loggerWithOpts$({
  silent: isTestEnv(),
  stream: createWriteStream(PATH, { flags: 'a' });;
  filter: req => req.status >= 400;
});
```
{% endcode %}

* **silent** - When `true` the logging is turned off \(usually useful during testing\),
* **stream** - Output stream for writing log messages, defaults to _process.stdout_. In the example above every response will be written to file pointed by provided `SOME_PATH` variable,
* **filter** - Function to determine if logging is skipped. For example we can log only HTTP status codes above _400_\).

