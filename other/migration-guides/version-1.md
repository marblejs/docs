---
description: >-
  This chapter provides a set of guidelines to help you migrate from Marble.js
  version 1.x to version 2.x.
---

# Migration from version 1.x

If you are new to Marble.js, starting with 2.x is easy-peasy. The biggest question for current 1.x users though, is how to migrate to the new version. You can ask, how many of the API has changed? Just relax — about 90% of the API is the same and the core concepts haven’t changed. 

## @marblejs/core

### Type declarations

**\#** `Effect`  👉 `HttpEffect`

\# `EffectResponse` 👉 `HttpEffectResponse`

**\#** `Middleware`   👉 `HttpMiddlewareEffect`

**\#** `ErrorEffect`  👉`HttpErrorEffect` 

### Effects - body, params, query

In order to improve request type inference HTTP Effect `req.params`, `req.body`, `req.query` are by default of `unknown` type instead of `any`.

**❌ Old:**

```typescript
const example$: Effect = (req$) =>
  req$.pipe(
    map(req => req.params.version),        // (typeof req.params) = any
    map(version => `Version: ${version}`), // (typeof version) = any
    map(message => ({ body: message })),
  );
```

**✅ New:**

```typescript
const example$: HttpEffect = (req$) =>
  req$.pipe(
    map(req => req.params.version),        // (typeof req.params) = unknown
    map(version => `Version: ${version}`), // ❌ type error!
    map(message => ({ body: message })),
  );
  
👇

const example$: HttpEffect = (req$) =>
  req$.pipe(
    map(req => req.params.version),        // (typeof req.params) = unknown
    map(version => version as string),     // or use validation middleware
    map(version => `Version: ${version}`), // ✅ looks fine!
    map(message => ({ body: message })),
  );
```

### Effects - third argument

From version 2.0 the effect function third argument is a common `EffectMetadata` object which can contain eventual error object or contextual injector.

**❌ Old:**

```typescript
const error$: ErrorEffect = (req$, _, error) =>
  req$.pipe(
    mapTo(error),
    // ...
  );
```

**✅ New:**

```typescript
const error$: HttpErrorEffect = (req$, _, meta) =>
  req$.pipe(
    mapTo(meta.error),
    // ...
  );
```

### httpListener error effect 

HTTP error effect is registered inside **httpListener** configuration object via `error$` instead of `errorEffect`.

**❌ Old:**

```typescript
export default httpListener({
  // ...
  errorEffect: // ...
});
```

**✅ New:**

```typescript
export default httpListener({
  // ...
  error$: // ...
});
```

### httpListener server bootstrapping

In order to use `httpListener` directly connected to ****Node.js `http.createServer` you have to run and apply Reader context first**.**

**❌ Old:**

```typescript
import { createContext } from '@marblejs/core';
import * as http from 'http';
import httpListener from './http.listener';
  
const server = http
  .createServer(httpListener)
  .listen(1337);
```

**✅ New:**

* **direct usage**

```typescript
import { createContext } from '@marblejs/core';
import * as http from 'http';
import httpListener from './http.listener';

const httpListenerWithContext = httpListener
  .run(createContext());
  
const server = http
  .createServer(httpListenerWithContext)
  .listen(1337);
```

* **using `createServer`** 

```typescript
import { createContext, createServer } from '@marblejs/core';
import httpListener from './http.listener';

const server = createServer({
  port: 1337,
  httpListener,
});

server.run();
```

## @marblejs/middleware-body

Every body parsing middleware should be registered via function invocation. It allows you to pass an optional middleware configuration object. 

**❌ Old:**

```typescript
import { bodyParser$ } '@marblejs/middleware-body';

httpListener({
  middlewares: [bodyParser$],
  // ...
});
```

**✅ New:**

```typescript
import { bodyParser$ } '@marblejs/middleware-body';

httpListener({
  middlewares: [bodyParser$()],
  // ...
});
```

## @marblejs/middleware-logger

Because previous `logger$` wasn't exposed as a function, it was very hard to extend it. Version 1.2.0 deprecated it in favor of `loggerWithOpts$` entry point which was more maintainable and could be extended easier. From the version **v2.x** the `logger$` entry point is be swapped with `loggerWithOpts$` implementation.

**❌ Old:**

```typescript
import { logger$, loggerWithOpts$ } '@marblejs/middleware-logger';

httpListener({
  middlewares: [
    logger$,
    // or
    loggerWithOpts$(),
  ],
  // ...
});
```

**✅ New:**

```typescript
import { bodyParser$ } '@marblejs/middleware-body';

httpListener({
  middlewares: [logger$()],
  // ...
});
```

