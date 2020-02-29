---
description: >-
  Every Marble.js listener factory allows you to intercept outgoing errors via
  dedicated error$ handler.
---

# Errors

## Building your own error effect

Middlewares and Effects are based on the same generic interface, so also error handlers can work in a very similar way.

```haskell
HttpErrorEffect :: Observable<{ req: HttpRequest; error: E }>
  -> Observable<HttpEffectResponse>
```

{% tabs %}
{% tab title="error.effect.ts" %}
```typescript
import { HttpErrorEffect } from '@marblejs/core';
import { map } from 'rxjs/operators';

const error$: HttpErrorEffect = req$ =>
  req$.pipe(
    map(({ req, error }) => ({
     status: error.status,
     body: error.data,
    }),
  );
```
{% endtab %}
{% endtabs %}

As any other Effect, error handler maps the stream of errored requests to objects of type HttpEffectResponse \(status, body, headers\). To connect the custom error handler, all you need to do is to attach it to `error$` property in `httpListener` config object.

{% hint style="info" %}
By default Marble.js comes with built-in error handler, but based on requirements you can override it anytime.
{% endhint %}

{% tabs %}
{% tab title="http.listener.ts" %}
```typescript
import { httpListener } from '@marblejs/core';
import { error$ } from './error.effect';

const listener = httpListener({
  middlewares,
  effects,
  error$ // 👈
});
```
{% endtab %}
{% endtabs %}

Lets take a look again at the previous example of authorization middleware. In case of unauthorized request, `authorize$` middleware will throw an error an propagate it to error stream \(custom or global one\).

{% tabs %}
{% tab title="auth.middleware.ts" %}
```typescript
import { HttpMiddlewareEffect, HttpError, HttpStatus } from '@marblejs/core';
import { of, throwError } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

const authorize$: HttpMiddlewareEffect = req$ =>
  req$.pipe(
    mergeMap(req => !isAuthorized(req),
      ? throwError(new HttpError('Unauthorized', HttpStatus.UNAUTHORIZED)),
      : of(req)),
  );
```
{% endtab %}
{% endtabs %}

Marble.js comes with a dedicated `HttpError` class for defining request related errors that can be caught easily inside error handler. Using RxJS built-in `throwError` function, we can throw an error and catch it on an upper level \(e.g. directly inside Effect or global error handler\).

The `HttpError` class resides in the `@marblejs/core` package. The constructor takes as a first parameter an error message and as a second parameter a `HttpStatus` code that can be a plain JavaScript number or TypeScript enum. Optionally you can pass the third argument, which can contain any other error related values.

```typescript
new HttpError('Forbidden', HttpStatus.FORBIDDEN, { resource: 'index.html' });
```

## 404 error handling

404 \(Not Found\) responses are not the result of an error, so the error handler will not capture them. This behavior is because a 404 response simply indicates the absence of matched Effect in request lifecycle; in other words, request didn't find a proper route. Since Marble.js comes with built-in effect for matching not-found routes, but you can override it anytime. All you need to do is to define a dedicated Effect at the very end of the effects stack to handle a 404 response.

{% tabs %}
{% tab title="not-found.effect.ts" %}
```typescript
import { r, HttpError, HttpStatus } from '@marblejs/core';
import { throwError } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

const notFound$ = r.pipe(
  r.matchPath('*'),
  r.matchType('*'),
  r.useEffect(req$ => req$.pipe(
    mergeMap(() =>
      throwError(new HttpError('Route not found', HttpStatus.NOT_FOUND))),
  )));
```
{% endtab %}
{% endtabs %}

The effect handles **all** paths and **all** method types then throws an 404 error code that can be intercepted inside global HttpErrorEffect.

"Not-found" Effects can be placed in any layer you want, eg. you can have multiple Effects that will handle missing routes in dedicated organization levels of your API structure.

