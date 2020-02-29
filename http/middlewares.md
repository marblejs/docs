---
description: >-
  In Marble.js, middlewares are streams of side-effects that can be composed and
  plugged-in to our request/event lifecycle to perform certain actions before
  reaching the designated Effect.
---

# Middlewares

## Building your own middleware

```haskell
MiddlewareEffect :: Observable<T> -> Observable<T>
```

Because everything here is a stream, the plugged-in middlewares are also based on a similar Effect interface.

```haskell
HttpMiddlewareEffect :: Observable<HttpRequest> -> Observable<HttpRequest>
```

By default, framework comes bundled with composable middlewares like: [logging](../other/api-reference/middleware-logger.md), request [body parsing](../other/api-reference/middleware-body.md) or request [validator](../other/api-reference/middleware-io.md). Below you can see how simple can look a dummy HTTP request logging middleware.

{% tabs %}
{% tab title="logger.middleware.ts" %}
```typescript
import { HttpMiddlewareEffect, HttpRequest } from '@marblejs/core';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

const logger$ = (req$: Observable<HttpRequest>): Observable<HttpRequest> =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${req.url}`)),
  );
  
// same as   
  
const logger$: HttpMiddlewareEffect = (req$, res) =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${req.url}`)),
  );  
```
{% endtab %}
{% endtabs %}

The example above performs I/O operation for every request that comes through the stream. As you can see, the middleware, compared to the basic effect, must return the processed request at the end.

In order to use our custom middleware, we need to attach the defined middleware to the `httpListener` config.

```typescript
import { httpListener } from '@marblejs/core';
import { logger$ } from './logger-middleware';

const middlewares = [
  // 👇 our custom middleware 
  logger$,
];

const listener = httpListener({ middlewares, effects });
```

### Parameterized middleware

There are some cases when our custom middleware needs to be parameterized - for example, the dummy `logger$` middleware should log request URL's conditionally. To achieve this behavior we can make our middleware function curried, where the last returned function should conform to`HttpMiddlewareEffect` interface.

```typescript
interface LoggerOpts {
  showUrl?: boolean;
}

const logger$ = (opts: LoggerOpts = {}): HttpMiddlewareEffect => req$ =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${opts.showUrl ? req.url : ''}`)),
  );
```

The improved logging middleware, can be composed like in the following example:

```typescript
const middlewares = [
  // 👇 our custom middleware
  logger$({ showUrl: true }),
];
```

### Sending a response earlier

Some types of middlewares need to send an HTTP response earlier. For this case Marble.js exposes a dedicated `req.res.send` method which allows to send an HTTP response using the same common interface that we use for sending a response inside API Effects. The mentioned method returns an empty Observable \(Observable that immediately completes\) as a result, so it can be composed easily inside a middleware pipeline.

```typescript
import { HttpMiddlewareEffect } from '@marblejs/core';
import { mergeMap } from 'rxjs/operators';

const middleware$: HttpMiddlewareEffect = req$ =>
  req$.pipe(
    mergeMap(req => req.res.send({ body: 💩, status: 304, headers: /* ... */ })),
  );
```

If the HTTP response is sent earlier than inside the target Effect, the execution of all following middlewares and Effects will be skipped.

## Middlewares composition

You can compose middlewares in four ways:

* globally \(inside `httpListener` configuration object\),
* inside grouped effects \(via `combineRoutes` function\),
* or by composing it directly inside Effect request pipeline.

### via Effect

There are many scenarios where we would like to apply middlewares inside our API Effects. One of them is to authorize only specific endpoints. Going to meet the requirements, Marble.js allows us to compose them using dedicated [use operator](../other/api-reference/core/operator-use.md), directly inside request stream pipeline.

Lets say we have an endpoint for getting list of all users, but also we would like to make it secure, and available only for authorized users. All we need to do is to compose authorization middleware using dedicated for this case `use` operator.

{% tabs %}
{% tab title="getUsers.effect.ts" %}
```typescript
import { use, r } from '@marblejs/core';
import { authorize$ } from './auth.middleware';

const getUsers$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  // 👇 here...
  r.use(authorize$),
  r.useEffect(req$ => req$.pipe(
    // 👇 or here...
    use(authorize$),
    // ...
  )));
```
{% endtab %}
{% endtabs %}

The example implementation of `authorize$` middleware can look like in the following snippet:

{% tabs %}
{% tab title="auth.middleware.ts" %}
```typescript
import { HttpMiddlewareEffect, HttpError, HttpStatus } from '@marblejs/core';
import { of, throwError } from 'rxjs';

const authorize$: HttpMiddlewareEffect = req$ =>
  req$.pipe(
    mergeMap(req => !isAuthorized(req),
      ? throwError(new HttpError('Unauthorized', HttpStatus.UNAUTHORIZED)),
      : of(req)),
  );
```
{% endtab %}
{% endtabs %}

### via combineRoutes

There are some cases where you would like to compose a bunch of middlewares before grouped routes, e.g. to authorize only a selected group of endpoints. Instead of composing middlewares for each route separately, you can also compose them via extended second parameter of `combineRoutes()` function.

```typescript
import { combineRoutes } from '@marblejs/core';

const api$ = combineRoutes('/api/v1', {
  middlewares: [ authorize$ ],
  effects: [ user$, movie$, actor$ ],
});
```

### via httpListener

If your middleware should operate globally, e.g. in case of request logging, the best place is to compose it inside `httpListener`. In this case the middleware will operate on each request that goes through your HTTP server.

```typescript
import { httpListener } from '@marblejs/core';
import { logger$ } from '@marblejs/middleware-body';
import { bodyParser$ } from '@marblejs/middleware-logger';

const middlewares = [
  logger$(),
  bodyParser$(),
];

const effects = [
  // ...
];

export const listener = httpListener({ middlewares, effects });
```

{% hint style="info" %}
The stacking order of middlewares inside `httpListener` and `combineRoutes` matters, because middlewares are run sequentially \(one after another\).
{% endhint %}

