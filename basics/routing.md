---
description: >-
  Routing refers to determining how an application responds to a client request
  to a particular endpoint, which is a path and a specific HTTP method (eg. GET,
  POST).
---

# Routing

## Route composition

Every API requires composable routing. With _Marble.js_ routing composition couldn't be easier. Lets assume that we have a separate **User** feature where its API endpoints respond to _GET_ and _POST_ methods on `/user` path.

In _Marble.js_ routing is builded via specialized `EffectFactory` function responsible for collecting information about _Effect_ routing details like path, request method type and connected _Effect_ handler.

{% code title="user.effects.ts" %}
```typescript
const getUsers$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // ...
  );

const postUser$: EffectFactory
  .matchPath('/')
  .matchType('POST')
  .use(req$ => req$.pipe(
    // ...
  );

export const user$ = combineRoutes(
  '/user',
  [ getUsers$, postUser$ ],
);
```
{% endcode %}

Each API _Effect_ can be grouped via  `combineRoutes` function which combines routing for different _Effects,_ prefixed with path passed as a first argument.

{% code title="api.effects.ts" %}
```typescript
import { user$ } from 'user.controller.ts';

const root$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // ...
  );

const foo$: EffectFactory
  .matchPath('/foo')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // ...
  );

export const api$ = combineRoutes(
  '/api/v1',
  [ root$, foo$, user$ ],
);
```
{% endcode %}

It is very important to mention that each combined _Effect_ group can be composed together, so as a result the routing can be built in much more structured manner. If we analyze example above, at a result it will be mapped to following API endpoints:

```text
GET    /api/v1
GET    /api/v1/foo
GET    /api/v1/user
POST   /api/v1/user
```

There are some cases when there is a need to compose a bunch of middlewares before grouped routes, eg. to authorize only a selecteted group of endpoints. Instead of composing middlewares for each route separately, using [use operator](../api-reference-old/use.md), you can also compose them via extended second parameter in`combineRoutes()` function:

```typescript
const api$ = combineRoutes('api/v1', {
  middlewares: [ authorize$ ],
  effects: [ user$ ],
});
```

## URL parameters

The `combineRoute` function and the _EffectFactory_ `matchPath` method allows you to define parameters in the path argument. All parameters are defined by syntax with a colon prefix. For example:

```typescript
EffectFactory
  .matchPath('/:foo/:bar')
  // ...
```

Parameters are localized in the request object via `req.params`. For above example and route `/bob/12`, the `req.params` object should contain following parameters:

```typescript
{
    foo: 'bob',
    bar: '12',
}
```

{% hint style="info" %}
You can validate URL parameters using dedicated [validator$](/marble/~/drafts/-LLENyDpOKSsz-gxDjG2/primary/middlewares/validatorusd) middleware.
{% endhint %}

Parameters can be suffixed with an asterisk \(`*`\) to denote a zero or more parameter matches. The code snippet below shows the example use case of "zero-or-more" parameter. It can be useful eg. for defining routing for static assets.

{% code title="getFile.effect.ts" %}
```typescript
const getFile$ = EffectFactory
  .matchPath('/:dir*')
  .matchType('GET')
  .use(req$ => req$.pipe(
    map(req => req.params.dir),
    switchMap(readFile(STATIC_PATH)),
    map(body => ({ body }))
  ));
```
{% endcode %}

## Query parameters

Except intercepting URL parameters, _EffectFactory_ `matchPath` method can also parse query parameters provided in path string. Query parameters are located inside `req.query` property, which is an object containing a property for each query string parameter in the route. If there is no query string, it is the empty object.

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
All properties and values in `req.query` object are untrusted and should be validated before trusting.
{% endhint %}

{% hint style="info" %}
You can validate incoming `req.query` parameters using dedicated [validator$](../available-middlewares/joi.md) middleware.
{% endhint %}

