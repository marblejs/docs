# Middlewares

It is a common requirement to encounter the necessity of having various operations needed around incoming HTTP requests to your server. In _Marble.js_ middlewares are streams of side-effects that can be composed and plugged-in to our request lifecycle to perform certain actions before reaching the designated _Effect_.

## Building your own middleware

Because everything here is a stream, also plugged-in middlewares are based on similar _Effect_ interface. By default, framework comes with composable middlewares like: [logging](../api-reference/middleware-logger.md), request [body parsing](../api-reference/middleware-body.md) or request [validator](../api-reference/middleware-io.md). Below you can see how simple can look the dummy HTTP request logging middleware.

```typescript
const logger$ = (req$: Observable<HttpRequest>, res: HttpResponse): Observable<HttpRequest> =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${req.url}`)),
  );
```

In the example above we get the stream of requests, then tap the `console.log` side effect and returns the same stream as a response of our middleware pipeline. The middleware, in comparison to basic effect, must return the request at the end.

If you prefer shorter type definitions, you can use a `HttpMiddlewareEffect` function interface.

```typescript
const logger$: HttpMiddlewareEffect = (req$, res) =>
  req$.pipe(
    // ...
  );
```

In order to use our custom middleware, waht we need to do is to attach the defined middleware to `httpListener` config.

```typescript
const middlewares = [
  // 👇 our custom middleware 
  logger$,
];

const app = httpListener({ middlewares, effects });
```

### Parametrized middleware

There are some cases when our custom middleware needs to be parametrized - for example dummy _logger$_ middleware should _console.log_  request URL's conditionally. To achieve this behavior we have to make our middleware function _curried_, where the last returned function should conform to`HttpMiddlewareEffect` interface.

```typescript
interface LoggerOpts {
  showUrl?: boolean;
}

const logger$ = (opts: LoggerOpts = {}): HttpMiddlewareEffect => req$ =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${opts.showUrl ? req.url : ''}`)),
  );
```

The improved logging middleare, can be composed like in the following example:

```typescript
const middlewares = [
  // 👇 our custom middleware
  logger$({ showUrl: true }),
];
```

### Sending a response earlier

Some types of middlewares need to send a HTTP response earlier. In this case Marble.js exposes a dedicated `res.send` method which allows to send a HTTP response using the same common interface that we use for sending a response inside API _Effects_. The mentioned method returns an empty _Observable_ \(_Observable that immediately completes_\) as a result, so it can be composed easily inside middleware pipeline.

```typescript
const middleware$: HttpMiddlewareEffect = (req$, res) =>
  req$.pipe(
    switchMap(() => res.send({ body: 💩, status: 304, headers: /* ... */ }),
  );
```

If the HTTP response is sent ealier than inside the target Effect, the execution of all following middlewares and Effects will be skipped.

## Middlewares composition

In _Marble.js_ you can compose middlewares in four ways:

* globally \(inside `httpListener` configuration object\),
* inside grouped effects \(via `combineRoutes` function\),
* or by composing it directly inside _Effect_ request pipeline.

### via _Effect_

There are many scenarrios where we would like to apply middlewares inside our API _Effects_. One of them is to authorize only specific endpoints. Going to meet the requirements, _Marble.js_ allows us to compose them using dedicated [use operator](../api-reference/core/operator-use.md), directly inside request stream pipeline.

Lets say we have an endpoint for getting list of all users registered in the system, but we would like to make it secure, and available only for authorized users. All we need to do is to compose authorization middleware using dedicated for this case `use` operator which takes as an argument our middleware.

{% code-tabs %}
{% code-tabs-item title="getUsers.effect.ts" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

Using `r.pipe` operators, the middlwares can be composed in two ways. The first one doesn't infer the returned `HttpRequest` type of chained middlewares.

The example implementation of `authorize$` middleware can look like in the following snippet:

{% code-tabs %}
{% code-tabs-item title="auth.middleware.ts" %}
```typescript
const authorize$: HttpMiddlewareEffect = req$ =>
  req$.pipe(
    mergeMap(req => iif(
      () => !isAuthorized(req),
      throwError(new HttpError('Unauthorized', HttpStatus.UNAUTHORIZED)),
      of(req),
    )),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
As you probably noticed, `auth.middleware` introduces an example use case of error handling. You can read more about it in dedicated [Error handling](error-handling.md) chapter.
{% endhint %}

### via _combineRoutes_

There are some cases where you have to compose a bunch of middlewares before grouped routes, eg. to authorize only a selecteted group of endpoints. Instead of composing middlewares for each route separately, using [use operator](../api-reference/core/operator-use.md), you can also compose them via extended second parameter in`combineRoutes()` function.

```typescript
const api$ = combineRoutes('api/v1', {
  middlewares: [ authorize$ ],
  effects: [ user$, movie$, actor$ ],
});
```

### via _httpListener_

If your middleware should operate globally, eg. in case of request logging, the best place is to compose it inside `httpListener`. In this case the middleware will operate for each request that will go through your HTTP server.

```typescript
const middlewares = [
  logger$(),
  bodyParser$(),
];

const effects = [
  // ...
];

export default httpListener({ middlewares, effects });
```

{% hint style="info" %}
The stacking order of middlewares inside `httpListener` and `combineRoutes` matters, because middlewares are run sequentially \(one after another\).
{% endhint %}



