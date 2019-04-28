# Middlewares

It is a common requirement to encounter the necessity of having various operations needed around incoming HTTP requests to your server. In _Marble.js_ middlewares are streams of side-effects that can be composed and plugged-in to our request lifecycle to perform certain actions before reaching the designated _Effect_.

## Building your own middleware

Because everything here is a stream, also plugged-in middlewares are based on similar _Effect_ interface. By default framework comes with composable middlewares like: [logging](../available-middlewares/logger.md), request [body parsing](../available-middlewares/body.md) or request [validator](../available-middlewares/joi.md). Below you can see how simple can look the dummy HTTP request logging middleware.

```typescript
const dummyLogger$: Effect<HttpRequest> = (req$, res) =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${req.url}`)),
  );
```

There are two important differences compared to API _Effects_:

* stream handler can take a `response` object as a second argument,
* middleware must return a stream of requests at the end of the pipeline.

If you prefer shorter type definitions, you can use a type alias which is a shorthand for `Effect<HttpRequest>` generic type:

```typescript
const dummyLogger$: Middleware = (req$, res) =>
  req$.pipe(
    // ...
  );
```

In the example above we get the stream of requests, tap `console.log` side effect and return the same stream as a response of our middleware pipeline. At the end, all we need to do is to attach the defined middleware to `httpListener` config.

```typescript
const middlewares = [
  // 👇 our custom middleware 
  dummyLogger$,
];

const app = httpListener({ middlewares, effects });
```

### Parametrized middleware

There are some cases when our custom middleware needs to be parametrized - for example dummy _logger$_ middleware should _console.log_  request URL's conditionally. To achieve this we need to **curry** our middleware by creating some kind of middleware factory, where the last returned function should conform to`Effect<HttpRequst>` generic interface.

```typescript
interface LoggerOptions {
  showUrl?: boolean;
}

const dummyLogger$ = (opts: LoggerOptions = {}): Middleware => req$ =>
  req$.pipe(
    tap(req => console.log(`${req.method} ${opts.showUrl ? req.url : ''}`)),
  );
```

Which can be composed like in the following example:

```typescript
const middlewares = [
  // 👇 our custom middleware
  dummyLogger$({ showUrl: true }),
];
```

### Sending a response earlier

Some types of middlewares need to send a HTTP response earlier. In this case Marble.js exposes a dedicated `req.send` method which allows to send a HTTP response using the same common interface that we use for sending a response inside API _Effects_. The method returns an empty _Observable_ \(_Observable that immediately completes_\) as a result, so it can be composed easily inside middleware pipeline.

```typescript
const middleware$: Middleware = (req$, res) =>
  req$.pipe(
    switchMap(() => res.send({ body: 💩, status: 304, headers: /* ... */ }),
  );
```

## Middlewares composition

In _Marble.js_ you can compose middlewares in three ways:

* globally \(inside `httpListener` configuration object\),
* inside grouped effects \(via `combineRoutes` function\),
* or by composing it directly inside _Effect_ request pipeline.

### via _Effect_

There are many case why we would like to apply middlewares inside our API _Effects_. One of them is to authorize only specific endpoints. Going to meet the requirements _Marble.js_ allows to do this using dedicated [use operator](../api-reference-old/use.md) responsible for composing middlewares directly inside request stream pipeline.

Lets say we have an endpoint for getting list of all users registered in the system, but we would like to make it secure, and available only for authorized users. All we need to do is to compose authorization middleware using dedicated for this case `use` operator which takes as an argument our middleware.

{% code-tabs %}
{% code-tabs-item title="getUsers.effect.ts" %}
```typescript
import { authorize$ } from 'auth.middleware`;

const getUsers$: EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // 👇 middleware composition 
    use(authorize$),
    // ...
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Where example implementation of `authorize$` middleware can look like in the following snippet:

{% code-tabs %}
{% code-tabs-item title="auth.middleware.ts" %}
```typescript
const authorize$: Middleware = req$ =>
  req$.pipe(
    switchMap(req => iif(
      () => !isAuthorized(req),
      throwError(new HttpError('Unauthorized', HttpStatus.UNAUTHORIZED)),
      of(req),
    )),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
As you probably noticed `auth.middleware` introduces a sample use case of error handling. You can read more about it in dedicated [Error handling](error-handling.md) chapter.
{% endhint %}

### via _combineRoutes_

There are some cases when there is a need to compose a bunch of middlewares before grouped routes, eg. to authorize only a selecteted group of endpoints. Instead of composing middlewares for each route separately, using [use operator](../api-reference-old/use.md), you can also compose them via extended second parameter in`combineRoutes()` function:

```typescript
const api$ = combineRoutes('api/v1', {
  middlewares: [ authorize$ ],
  effects: [ user$ ],
});
```

### via _httpListener_

If your the middleware should operate globally, eg. in case of request logging, then the best place to compose it inside `httpListener`. In this case the middleware will operate for each request that will go through your HTTP server.

```typescript
const middlewares = [
  logger$,
  bodyParser$,
];

const effects = [...];

export const app = httpListener({ middlewares, effects });
```

{% hint style="info" %}
The order of placing middlewares inside `httpListener` and `combineRoutes` matters, because the middlewares are run sequentially \(one after another\). 
{% endhint %}

## Available middlewares

You can check all available middlewares here:

{% page-ref page="../available-middlewares/" %}



