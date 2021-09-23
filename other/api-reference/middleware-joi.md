---
description: A Joi validation middleware for Marble.js.
---

# @marblejs-contrib/middleware-joi

![](../../.gitbook/assets/68747470733a2f2f7261772e6769746875622e636f6d2f686170696a732f6a6f692f6d61737465722f696d616765732f6a6f692e706e67.png)

[Joi](https://github.com/hapijs/joi) is an object schema description language and validator for JavaScript objects. Using its schema language, you can validate things like:

* HTTP request headers
* HTTP body parameters
* HTTP request query parameters
* URL parameters

You can find detailed API reference for Joi schemas [here](https://github.com/hapijs/joi/blob/v13.6.0/API.md).

{% hint style="warning" %}
**Deprecation warning**

Since version 4.0 `@marblejs-contrib/middleware-joi` package is deprecated and won't be maintained anymore.
{% endhint %}

{% hint style="warning" %}
Since version 4.0, the middleware is a part of contrib packages. If you really want to use this middleware you can reach it via `@marblejs-contrib/middleware-joi`.
{% endhint %}

{% hint style="warning" %}
For more advanced request or event validation purposes we highly recommend to use [@marblejs/middleware-io](middleware-io.md) package instead. It can better handle type inference for complex schemas.
{% endhint %}

### Installation

```bash
yarn add @marblejs-contrib/middleware-joi
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { validator$ } from '@marblejs-contrib/middleware-joi';
```

### Type declaration

```text
validator$ :: (Schema, Joi.ValidationOptions) -> HttpMiddlewareEffect
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _schema_ | `Schema` |
| _options_ | &lt;optional&gt; `Joi.ValidationOptions` \(see: [Joi API reference](https://github.com/hapijs/joi/blob/v13.6.0/API.md#joi)\) |

#### _**Schema**_

| _parameter_ | definition |
| :--- | :--- |
| _headers_ | &lt;optional&gt; `any` |
| _params_ | &lt;optional&gt; `any` |
| _query_ | &lt;optional&gt; `any` |
| _body_ | &lt;optional&gt; `any` |

### Usage

**1.** Example of using middleware on a _GET_ route to validate query parameters:

{% code title="foo.effect.ts" %}
```typescript
import { r } from '@marblejs/http';
import { validator$, Joi } from '@marblejs-contrib/middleware-joi';

const foo$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    use(validator$({
      query: Joi.object({
        id: Joi.number().min(1).max(10),
      })
    }));
    // ...
  )));
```
{% endcode %}

Example above will validate each incoming request connected with `foo$` _Effect_. The validation blueprint defines that the `id` query parameter should be a number between _&lt;1..10&gt;_. If the schema requirements are not satisfied the middleware will throw an error with description what went wrong.

```typescript
{
  error: {
    status: 400,
    message: '"id" is required'
  }
}
```

**2.** Example of validating all incoming requests:

{% code title="app.ts" %}
```typescript
import { validator$, Joi } from '@marblejs-contrib/middleware-joi';

const middlewares = [
  // ...
  validator$(
    {
      headers: Joi.object({
        token: Joi.string().token().required(),
        accept: Joi.string().default('application/json')
      })
    },
    { stripUnknown: true },
  )
];

const effects = [
  endpoint1$,
  endpoint2$,
  ...
];

const app = httpListener({ middlewares, effects });
```
{% endcode %}

### Credits

Middleware author: [Lucio Rubens](https://github.com/luciorubeens)

