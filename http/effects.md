---
description: >-
  Effect is the main building block of the whole framework. It is just a
  function that returns a stream of events. Using its generic interface we can
  define API endpoints, event handlers or middlewares.
---

# Effects

## Building your own Effect

```haskell
Effect :: Observable<T> -> Observable<U>
```

Marble.js **HttpEffect** is a more specialized form of Effect for processing HTTP requests. Its responsibility is to map every incoming request to response object.

```haskell
HttpEffect :: Observable<HttpRequest> -> Observable<HttpEffectResponse>
```

```typescript
import { HttpEffect, HttpEffectResponse, HttpRequest } from '@marblejs/http';
import { Observable } from 'rxjs/operators';
import { mapTo } from 'rxjs/operators';

const hello$ = (req$: Observable<HttpRequest>): Observable<HttpEffectResponse> =>
  req$.pipe(
    mapTo({ body: 'Hello, world!' }),
  );

// same as 

const hello$: HttpEffect = req$ =>
  req$.pipe(
    mapTo({ body: 'Hello, world!' }),
  );
```

The Effect above responds to incoming request with _"Hello, world!"_ message. In case of HTTP protocol each incoming request has to be mapped to an object with **body**, **status** or **headers** attributes. If the status code or headers are not defined, then the API by default responds with `200 OK` status and `Content-Type: application/json` header.

{% hint style="info" %}
Every Marble.js Effect is eagerly bootstrapped on app startup with its own hot Observable.  
It means that a function can act as a constructor and a returned stream as a place "where the magic happens". It gives you a set of possibilities in terms of optimization, so e.g. you can inject all required [context readers](context.md) during the app startup only once. 
{% endhint %}

In order to route our first HttpEffect_,_ we have to define the path and method that the incoming request should be matched to. The simplest implementation of an HTTP API endpoint can look like this.

{% tabs %}
{% tab title="hello.effect.ts" %}
```typescript
import { r } from '@marblejs/http';
import { mapTo } from 'rxjs/operators';

const hello$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    mapTo({ body: 'Hello, world!' }),
  )));
```
{% endtab %}
{% endtabs %}

Let's define a little bit more complex endpoint.

{% tabs %}
{% tab title="postUser.effect.ts" %}
```typescript
import { r } from '@marblejs/http';
import { map, mergeMap } from 'rxjs/operators';
import { User, createUser } from './user.helper';

const postUser$ = r.pipe(
  r.matchPath('/user'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    map(req => req.body as User),
    mergeMap(createUser),
    map(body => ({ body }))
  )));
```
{% endtab %}
{% endtabs %}

The example above will match every POST request that matches to `/user` url. Using previously parsed body \(see [bodyParser$](../other/api-reference/middleware-body.md) middleware\) we can flat map it to other stream and map again to `HttpEffectResponse` object as an action confirmation.

{% hint style="warning" %}
**Deprecation warning**

With an introduction of Marble.js 4.0, old [`EffectFactory`]() HTTP route builder does not exist anymore. Please use[`r.pipe`](../other/api-reference/marblejs-http/r.pipe.md) builder instead.
{% endhint %}

**HttpRequest**

Every HttpEffect has an access to two most basics objects created by http.Server. **HttpRequest** is an abstraction over the basic _Node.js_ [http.IncomingMessage](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage) object. It may be used to access response status, headers and data, but in most scenarios you don't have to deal with all available APIs offered by IncomingMessage class. The most common properties available in request object are:

* url
* method
* headers
* body \(see [bodyParser$](../other/api-reference/middleware-body.md) section\)
* params \(see [routing](routing.md) chapter\)
* query \(see [routing](routing.md) chapter\)
* res \(**HttpResponse\)**

{% hint style="info" %}
For more details about available API offered in `http.IncomingMessage`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage).
{% endhint %}

**HttpResponse**

Response object is also an abstraction over basic _Node.js_ [http.ServerResponse](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse) object. Besides the default API, the response object exposes an `res.send` method, which can be a handy wrapper over _Marble.js_ responding mechanism. For more information about the _res.send_ method, visit [Middlewares](middlewares.md) chapter.

{% hint style="warning" %}
Since Marble.js version 3.0, HttpResponse object is accessible via `req.res` attribute of every HttpRequest.
{% endhint %}

{% hint style="info" %}
For more details about the available API offered in `http.ServerResponse`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse).
{% endhint %}

