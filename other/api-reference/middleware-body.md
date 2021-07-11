---
description: HTTP request body parser middleware for Marble.js.
---

# middleware-body

### Installation

```bash
yarn add @marblejs/middleware-body
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { bodyParser$ } from '@marblejs/middleware-body';
```

### Type declaration <a id="type-declaration"></a>

```haskell
bodyParser$ :: BodyParserOptions -> HttpMiddlewareEffect
```

### Parameters

| parameter | definition |
| :--- | :--- |
| _options_ | &lt;optional&gt; `BodyParserOptions` |

_**BodyParserOptions**_

| _**parameter**_ | definition |
| :--- | :--- |
| _parser_ | &lt;optional&gt; `RequestBodyParser` |
| _type_ | &lt;optional&gt; `Array<string>` |

The `type` option is used to determine what media type the middleware will parse. It is passed directly to the [type-is](https://www.npmjs.org/package/type-is#readme) library and this can be an extension name \(like `json`\), a mime type \(like `application/json`\), or a mime type with a wildcard \(like `*/*` or `*/json`\). Defaults to `*/*`.

### Basic usage

{% code title="app.ts" %}
```typescript
import { bodyParser$ } from '@marblejs/middleware-body';

export default httpListener({
  middlewares: [bodyParser$()],
  effects: [/* ... */],
});
```
{% endcode %}

Lets assume that we have the following _CURL_ command, which triggers `POST /api/login` endpoint:

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

The _POST_ request body can be intercepted like follows.

{% code title="login.effect.ts" %}
```typescript
import { r } from '@marblejs/core';

export const login$ = r.pipe(
  r.matchPath('/login'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    map(req => req.body as { username: string, password: string }),
    map(body => ({ body: `Hello, ${body.username}!` }))
  )));
```
{% endcode %}

{% hint style="danger" %}
All properties and values in `req.body` object are untrusted \(unknown\) and should be validated before trusting.
{% endhint %}

{% hint style="info" %}
This middleware does not handle multipart bodies.
{% endhint %}

### Advanced usage

The middleware does nothing if request _Content-Type_ is not matched, which makes a possibility for chaining multiple parsers one after another. For example, we can register multiple middlewares that will parse only a certain set of possible _Content-Type_ headers. 

**Default parser:**

```typescript
import { bodyParser$ } from '@marblejs/middleware-body';

bodyparser$();
```

Parses _application/json_ \(to JSON\), _application/x-www-form-urlencoded_ \(to JSON\), _application/octet-stream_ \(to Buffer\) and _text/plain_ \(to string\) content types

**JSON parser:**

```typescript
import { bodyParser$, jsonParser } from '@marblejs/middleware-body';

bodyParser$({
  parser: jsonParser,
  type: ['*/json', 'application/vnd.api+json'],
})
```

Parses `req.body` to JSON object.

**URLEncoded parser:**

```typescript
import { bodyParser$, urlEncodedParser } from '@marblejs/middleware-body';

bodyParser$({
  parser: urlEncodedParser,
  type: ['*/x-www-form-urlencoded'],
});
```

Parses encoded URLs `req.body` to JSON object.

**Text parser:**

```typescript
import { bodyParser$, textParser } from '@marblejs/middleware-body';

bodyParser$({
  parser: textParser,
  type: ['text/*'],
});
```

Parses `req.body` to string.

**Raw parser:**

```typescript
import { bodyParser$, rawParser } from '@marblejs/middleware-body';

bodyParser$({
  parser: rawParser,
  type: ['application/octet-stream'],
}),
```

Parses `req.body` to Buffer.

### Custom parsers

If the available parsers are not enough, you can create your own body parsers by conforming to `RequestBodyParser` interface. The example below shows how the `jsonParser` looks underneath.

```typescript
export const jsonParser: RequestBodyParser = req => body =>
  JSON.parse(body.toString());
```

