---
description: HTTP request body parser middleware for Marble.js.
---

# middleware-body

### Installation

```bash
$ npm i @marblejs/middleware-body
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { bodyParser$ } from '@marblejs/middleware-body';
```

### Usage

{% code-tabs %}
{% code-tabs-item title="app.ts" %}
```typescript
import { bodyParser$ } from '@marblejs/middleware-body';

const middlewares = [
  bodyParser$,
  ...
];

const effects = [
  ...
];

export const app = httpListener({ middlewares, effects });
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Lets assume that we have the following _CURL_ command, which triggers `POST api/login` endpoint:

```bash
$ curl --header "Content-Type: application/json" \
  --request POST \
  --data '{ "username": "foo", "password": "bar" }' \
  http://localhost:3000/api/login
```

Using previously connected `bodyParser$`  middleware, the app will intercept the following payload object:

```typescript
req.body = {
  username: 'foo',
  password: 'bar',
};
```

where the _POST_ request body can be intercepted inside sample _Effect_ like follows:

{% code-tabs %}
{% code-tabs-item title="dummyLogin.effect.ts" %}
```typescript
const dummyLogin$ = EffectFactory
  .matchPath('/login')
  .matchType('POST')
  .use(req$ => req$.pipe(
    map(req => req.body)
    map(body => ({ body: `Hello, ${body.username}!` }))
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="danger" %}
All properties and values in `req.body` object are untrusted and should be validated before trusting.
{% endhint %}

{% hint style="info" %}
This middleware does not handle multipart bodies.
{% endhint %}



