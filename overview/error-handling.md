# Error handling

## Error handling

Lets take a look again at the previous example of authorization middleware.

{% code-tabs %}
{% code-tabs-item title="auth.middleware.ts" %}
```typescript
import { HttpError, HttpStatus } from '@marblejs/core';

const authorize$: HttpMiddlewareEffect = req$ =>
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

_Marble.js comes with dedicated `HttpError`_ class for defining a request related errors that can be catched easily via error middleware. Using _RxJS_ built-in `throwError` method, we can throw an error and catch it on an upper level \(eg. directly inside _Effect_ or _ErrorEffect_\).

`HttpError` class resides in `@marblejs/core` package. The constructor takes as a first parameter an error message and as a second parameter a `HttpStatus` code that can be a plain _JavaScript_ `number` or _TypeScript_ enum type that can also be imported via core package. Optionally you can pass the third argument, which can contain any other error related values.

```typescript
new HttpError('Forbidden', HttpStatus.FORBIDDEN, { resource: 'index.html' });
```

## 404 error handling

_404 \(Not Found\)_ responses are not the result of an error, so the error handler will not capture them. This behavior is because a _404_ response simply indicates the absence of matched _Effect_ in request lifecycle; in other words, request didn't find a proper route. All you need to do is to define a dedicated _Effect_ at the very end of the effects stack to handle a 404 response.

```typescript
const notFound$ = r.pipe(
  r.matchPath('*'),
  r.matchType('*'),
  r.useEffect(req$ => req$.pipe(
    mergeMap(() =>
      throwError(new HttpError('Route not found', HttpStatus.NOT_FOUND))
    )
  )));
```

The _HttpEffect_ above handles **all** paths and **all** method types ****and throws an 404 error code that can be intercepted via _HttpErrorEffect_.

`notFound$` _Effects_ can be placed in any layer you want, eg. you can have multiple _Effects_ that will handle missing routes in dedicated organization levels of your API structure.

## Custom error handling effect

By default _Marble.js_ comes with simple and lightweight error handling _Effect_. Because middlewares and _Effects_ are based on the same generic interface, your error handlers can work very similar.

{% code-tabs %}
{% code-tabs-item title="error.middleware.ts" %}
```typescript
const customError$: HttpErrorEffect<ThrownError> = (req$, res, meta) =>
  req$.pipe(
    mapTo(meta.error),
    map(error => ({
     status: error.status
     body: error.data
    }),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

As any other _Effect_, error handler maps the stream of errored requests to objects of type _HttpEffectResponse_ \(`status`, `body`, `headers`\). The _HttpErrorEffect_ can retrieve from the third argument an intercepted error object which can be used for error handling-related logic.

To connect the custom error handler, all you need to do is to attach it to `error$` property in `httpListener` config object.

{% code-tabs %}
{% code-tabs-item title="http.listener.ts" %}
```typescript
httpListener({
  middlewares,
  effects,
  // Custom error effect 👇
  error$: customError$,
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}



