# Error handling

## Error handling

Lets take a look again at sample authorization middleware.

{% code-tabs %}
{% code-tabs-item title="auth.middleware.ts" %}
```typescript
export const authorize$: Middleware = req$ =>
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

_Marble.js comes with dedicated `HttpError`_ class for defining a request related errors that can be catched easily via error middleware. Using _RxJS_ built-in `throwError` method, we can throw a passed error object and catch it on an upper level \(eg. inside API Effect or error middleware\) via _RxJS_ `catchError`method.

`HttpError` class resides in `@marblejs/core` package and its constructor takes as a first parameter an error message and as a second parameter a `HttpStatus` code that can be a plain _JavaScript_ `number` or _TypeScript_ enum type that also can be imported via core package. Optionally you can pass also a `data` object for any other error related datas as a third argument.

```typescript
new HttpError('Forbidden', HttpStatus.FORBIDDEN, { resource: 'index.html' });
```

## 404 error handling

_404 \(Not Found\)_ responses are not the result of an error, so the error-handler middleware will not capture them. This behavior is because a _404_ response simply indicates the absence of matched _Effect_ in request lifecycle; in other words, request didn't find a proper route. All you need to do is to compose a dedicated _Effect_ at the very end of the stack to handle a 404 response.

```typescript
const notFound$ = EffectFactory
  .matchPath('*')
  .matchType('*')
  .use(req$ => req$.pipe(
    switchMap(() =>
      throwError(new HttpError('Route not found', HttpStatus.NOT_FOUND))
    )
  ));
```

The _Effect_ above will handle **all** paths and **all** method types ****and at the end of pipeline will throw an 404 error code that can be intercepted via error middleware.

`notFound$` _Effects_ can be placed in any layer you want, eg. you can have multiple _Effects_ that will handle missing routes in dedicated organization levels of your API structure.

## Custom error handling effect

By default _Marble.js_ comes with simple and lightweight error handling effect. Because _middlewares_ and _Effects_ are based on the same generic interface, your error handlers can work very similar to normal API _Effects_.

{% code-tabs %}
{% code-tabs-item title="error.middleware.ts" %}
```typescript
const error$: Effect<EffectResponse, ThrownError> = (req$, res, err) =>
  req$.pipe(
     map(req => ({
       status: // ...
       body:  // ...
     }),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

As any other _Effect_, error handler maps the stream of errored requests to objects of type _EffectResponse_ \(`status`, `body`, `headers`\). The difference is that it takes as a third argument an intercepted error object which can be used for error handling-related logic.

To connect the custom error handler, all you need to do is to attach it to `errorEffect` property in `httpListener` config object.

{% code-tabs %}
{% code-tabs-item title="app.ts" %}
```typescript
const app = httpListener({
  middlewares,
  effects,
  // Custom error effect 👇
  errorEffect: error$,
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}



