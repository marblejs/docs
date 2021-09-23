---
description: >-
  Routing determines how an application responds to a client request to a
  particular endpoint, which is a path and a specific HTTP method (eg. GET,
  POST).
---

# Routing

## Route composition

As we know - every API requires composable routing. Lets assume that we have a separate **User** feature where its API endpoints respond to GET and POST methods on `/user` path.

{% hint style="warning" %}
**Deprecation warning**

With an introduction of Marble.js 4.0, old [`EffectFactory`]() HTTP route builder does not exists anymore. Please use[`r.pipe`](../other/api-reference/marblejs-http/r.pipe.md) builder instead.
{% endhint %}

`r.pipe` is an indexed monad builder used for collecting information about Marble REST route details, like: path, request method type, middlewares and connected Effect.

{% tabs %}
{% tab title="user.effects.ts" %}
```typescript
import { combineRoutes, r } from '@marblejs/http';

const getUsers$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    // ...
  )));

const postUser$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    // ...
  )));

export const user$ = combineRoutes('/user', [
  getUsers$,
  postUser$,
]);
```
{% endtab %}
{% endtabs %}

Route effects can be grouped together using `combineRoutes` function, which combines routing for a prefixed path passed as a first argument. Exported group of Effects can be combined with other Effects like in the example below.

{% tabs %}
{% tab title="api.effects.ts" %}
```typescript
import { combineRoutes, r } from '@marblejs/http';
import { user$ } from './user.effects';

const root$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    // ...
  )));

const foo$ = r.pipe(
  r.matchPath('/foo'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    // ...
  )));

export const api$ = combineRoutes('/api/v1', [
  root$,
  foo$,
  user$, // 👈
]);
```
{% endtab %}
{% endtabs %}

As you can see, the previously defined routes can be combined together, so as a result the routing is built in a much more structured way. If we analyze the above example, the routing will be mapped to the following routing table.

```text
GET    /api/v1
GET    /api/v1/foo
GET    /api/v1/user
POST   /api/v1/user
```

There are some cases where there is a need to compose a bunch of middlewares before grouped routes, e.g. to authenticate requests only for a selected group of endpoints. Instead of composing middlewares using [use operator](../other/api-reference/core/operator-use.md) for each route separately, you can compose them via the extended second parameter in`combineRoutes()` function.

```typescript
import { combineRoutes } from '@marblejs/http';

const user$ = combineRoutes('/user', {
  middlewares: [authorize$],
  effects: [getUsers$, postUser$],
});
```

## Body parameters

Marble.js doesn't come with a built-in mechanism for parsing POST, PUT and PATCH request bodies. In order to get the parsed request body you can use dedicated _@marblejs/middleware-body_ package. A new `req.body` object containing the parsed data will be populated on the request object after the middleware, or undefined if there was no body to parse, the `Content-Type` was not matched, or an error occurred. To learn more about body parsing middleware visit the [@marblejs/middleware-body](../other/api-reference/middleware-body.md) API specification.

{% hint style="danger" %}
All properties and values in `req.body`object are untrusted and should be validated before usage.
{% endhint %}

{% hint style="danger" %}
By design, the `req.body, req.params, req.query`are of type `unknown`. In order to work with decoded values you should validate them before \(e.g. using dedicated validator middleware\) or explicitly assert attributes to the given type. We highly recommend to use the [@marblejs/middlware-io](../other/api-reference/middleware-io.md) package which allows you to properly infer the type of validated properties.
{% endhint %}

## URL parameters

The `combineRoutes` function and the `r.matchPath` allows you to define parameters in the path argument. All parameters are defined by the syntax with a colon prefix.

```typescript
import { r } from '@marblejs/http';

const foo$ = r.pipe(
  r.matchPath('/:foo/:bar'),
  // ...
);
```

Decoded path parameters are placed in the `req.params` property. If there are no decoded URL parameters then the property contains an empty object. For the above example and route `/bob/12` the `req.params` object will contain the following properties:

```typescript
{
  foo: 'bob',
  bar: '12',
}
```

For parsing and decoding URL parameters, Marble.js makes use of [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp) library.

{% hint style="danger" %}
All properties and values in `req.params` object are untrusted and should be validated before usage.
{% endhint %}

{% hint style="info" %}
You should validate incoming URL params using dedicated [requestValidator$](../other/api-reference/middleware-io.md) middleware.
{% endhint %}

Path parameters can be suffixed with an asterisk \(`*`\) to denote a zero or more parameter matches. The code snippet below shows an example use case of a "zero-or-more" parameter. For example, it can be useful for defining routing for static assets.

{% tabs %}
{% tab title="getFile.effect.ts" %}
```typescript
import { r } from '@marblejs/http';
import { map, mergeMap } from 'rxjs/operators';

const getFile$ = r.pipe(
  r.matchPath('/:dir*'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    // ...
    map(req => req.params.dir),
    mergeMap(readFile(STATIC_PATH)),
    map(body => ({ body }))
  )));
```
{% endtab %}
{% endtabs %}

## Query parameters

Except intercepting URL params, the routing is able to parse query parameters provided in path string. All decoded query parameters are located inside `req.query` property. If there are no decoded query parameters then the property contains an empty object. For parsing and decoding query parameters, Marble.js makes use of [`qs`](https://github.com/ljharb/qs) library.

**Example 1:**

```text
GET /user?name=Patrick
```

```typescript
req.query = {
  name: 'Patrick',
};
```

**Example 2:**

```text
GET /user?name=Patrick&location[country]=Poland&location[city]=Katowice
```

```typescript
req.query = {
  name: 'Patrick',
  location: {
    country: 'Poland',
    city: 'Katowice',
  },
};
```

{% hint style="danger" %}
All properties and values in `req.query` object are untrusted and should be validated before usage.
{% endhint %}

{% hint style="info" %}
You should validate incoming `req.query` parameters using dedicated[ requestValidator$](../other/api-reference/middleware-io.md) middleware.
{% endhint %}

