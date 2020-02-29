---
description: HTTP requests authentication middleware for Marble.js based on JWT mechanism.
---

# middleware-jwt

![](../../.gitbook/assets/1-0g_7ab6zzumee-rdjngjkq.png)

This module lets you authenticate HTTP requests using JWT tokens in your **Marble.js** applications. JWTs are typically used to protect API endpoints, and are often issued using OpenID Connect.

{% hint style="info" %}
You can find more details about JWT \(RFC 7519\) standard [here](http://jwt.io).
{% endhint %}

The middleware uses `jsonwebtoken` package under the hood. It wraps the package API into more  
RxJS-friendly abstractions that can be partially applied and composed inside Effect streams.

### Installation

{% code title="" %}
```bash
$ npm i @marblejs/middleware-jwt
```
{% endcode %}

Requires `@marblejs/core` to be installed.

### Importing

```typescript
import { authorize$ } from '@marblejs/middleware-jwt';
```

### Type declaration

```text
authorize$ :: (VerifyOptions, object -> Observable<object>) -> Middleware
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _config_ | `VerifyOptions` |
| _verifyPayload$_ | `(payload: object) => Observable<object>` |

{% tabs %}
{% tab title="verifyPayload$" %}
A function used for payload verification. With this handler we can check if the payload extracted from the JWT token fulfills our security criterias \(eg. we can verify if the given user identifier exists in the database\). Besides the general verification, the function can return the streamed object, that will be available inside _HttpRequest_ `req.user` parameter. If the stream throws an error \(eg. during the validation\) the _authorize$_ middleware responds with `401 / Unauthorized` error.

**Type declaration**

```text
verifyPayload$ :: object -> Observable<object>
```

**Example**

The function checks the presence of the user id in the database and then returns an _Observable_ of found user instance. 

{% code title="" %}
```typescript
const verifyPayload$ = (payload: Payload) => UserDao
  .findById(payload._id)
  .pipe(flatMap(neverNullable));
```
{% endcode %}
{% endtab %}

{% tab title="VerifyOptions" %}
Config object, passed as a first argument, defines a set of parameters that are used during token verification process.

| _parameter_ | definition |
| :--- | :--- |
| _secret_ | `string | Buffer` |
| algorithms | &lt;optional&gt;  `string[]` |
| audience | &lt;optional&gt; `string | string[]` |
| clockTimestamp | &lt;optional&gt; `number` |
| clockTolerance | &lt;optional&gt; `number` |
| issuer | &lt;optional&gt; `string | string[]` |
| ignoreExpiration | &lt;optional&gt; `boolean` |
| ignoreNotBefore | &lt;optional&gt; `boolean` |
| jwtid | &lt;optional&gt; `string` |
| subject | &lt;optional&gt; `string` |
{% endtab %}
{% endtabs %}

{% hint style="info" %}
For more infos about _jwt.VerifyOptions_ please read [jsonwebtoken docs](https://github.com/auth0/node-jsonwebtoken#jwtverifytoken-secretorpublickey-options-callback).
{% endhint %}

{% hint style="info" %}
You can read more about token creation [here](token-creation.md).
{% endhint %}

### **Usage**

It is recommended to extract the middleware configuration into separate file. This way you can reuse it in many places.

{% code title="auth.middleware.ts" %}
```typescript
import { authorize$ as jwt$ } from '@marblejs/middleware-jwt';
import { SECRET_KEY } from './config';

const config = { secret: SECRET_KEY };

const verifyPayload$ = (payload: Payload) => UserDao
  .findById(payload._id)
  .pipe(flatMap(neverNullable));

export const authorize$ = jwt$(config, verifyPayload$);
```
{% endcode %}

The configured middleware can be simply composed in any route, that should be validated.

```typescript
import { EffectFactory } from '@marblejs/core';
import { authorize$ } from './auth-middleware';

const getUsers$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(getUsersEffect$);

const user$ = combineRoutes('/user', {
  effects: [getUsers$],
  middlewares: [authorize$], 👈
});
```

If the incoming request doesn't pass the authentication process \(eg. token is invalid, expired or the `verifyPayload$` throws an error, the middleware responds with `401 / Unauthorized` error.



