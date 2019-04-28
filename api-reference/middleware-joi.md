---
description: A Joi validation middleware for Marble.js.
---

# middleware-joi

![](../.gitbook/assets/68747470733a2f2f7261772e6769746875622e636f6d2f686170696a732f6a6f692f6d61737465722f696d616765732f6a6f.png)

[Joi](https://github.com/hapijs/joi) is an object schema description language and validator for JavaScript objects. Using its schema language, you can validate things like:

* HTTP request headers
* HTTP body parameters
* HTTP request query parameters
* URL parameters

You can find detailed API reference for Joi schemas [here](https://github.com/hapijs/joi/blob/v13.6.0/API.md).

{% hint style="warning" %}
For more advanced request or event validation purposes we highly recommend to use [@marblejs/middleware-io](middleware-io.md) package instead. It can better handle type inference for complex schemas.
{% endhint %}

### Installation

```bash
$ npm i @marblejs/middleware-joi
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { validator$ } from '@marblejs/middleware-joi';
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

{% code-tabs %}
{% code-tabs-item title="foo.effect.ts" %}
```typescript
import { r } from '@marblejs/core';
import { validator$, Joi } from '@marblejs/middleware-joi';

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
{% endcode-tabs-item %}
{% endcode-tabs %}

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

{% code-tabs %}
{% code-tabs-item title="app.ts" %}
```typescript
import { validator$, Joi } from '@marblejs/middleware-joi';

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
{% endcode-tabs-item %}
{% endcode-tabs %}

### Credits

Middleware author: [Lucio Rubens](https://github.com/luciorubeens)

