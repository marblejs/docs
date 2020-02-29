---
description: >-
  Besides the common things like token authorization, the middleware comes with
  handy functions responsible for token signing.
---

# Token signing

## + **generateToken**

The middleware wraps auth0 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) API into more RxJS friendly functions that can be partially applied and composed inside Observable streams.

**generateToken** signs new JWT token with provided payload and configuration object which defines the way how the token is signed. 

### Importing

```typescript
import { generateToken } from '@marblejs/middleware-jwt';
```

### Type declaration

```text
generateToken :: GenerateOptions -> Payload -> string
```

### Parameters

| _parameter_ | definition |
| :--- | :--- |
| _options_ | `GenerateOptions` |
| _payload_ | `Payload = string | object | Buffer` |

{% tabs %}
{% tab title="GenerateOptions" %}
Config object which defines a set of parameters that are used for token signing.

| _parameter_ | definition |
| :--- | :--- |
| _secret_ |  `string | Buffer` |
| algorithm | &lt;optional&gt; `string` |
| keyid | &lt;optional&gt; `string` |
| expiresIn | &lt;optional&gt; `string | number` |
| notBefore | &lt;optional&gt; `string | number` |
| audience | &lt;optional&gt; `string | string[]` |
| subject | &lt;optional&gt; `string` |
| issuer | &lt;optional&gt; `string` |
| jwtid | &lt;optional&gt; `string` |
| noTimestamp | &lt;optional&gt; `boolean` |
| header | &lt;optional&gt; `object` |
| encoding | &lt;optional&gt; `string` |
{% endtab %}
{% endtabs %}

{% hint style="info" %}
For more details about JWT token signing, please visit [jsonwebtoken package docs](https://github.com/auth0/node-jsonwebtoken).
{% endhint %}

## + **generateExpirationInHours**

The standard for JWT defines an `exp` claim for expiration. The expiration is represented as a **NumericDate.** This means that the expiration should contain the number of seconds since the epoch.

**generateExpiratinoInHours** is a small, but handy function that returns an numeric date for given hours as a parameter. If the function is called without any parameter then the date is generated with 1 hour expiration.

### Importing

```typescript
import { generateExpirationInHours } from '@marblejs/middleware-jwt';
```

### Type declaration

```text
generateExpirationInHours :: number -> number
```

## **Example**

{% code title="token.helper.ts" %}
```typescript
export const generateTokenPayload = (user: User) => ({
  id: user.id,
  email: user.email,
  exp: generateExpirationInHours(4), 
  // 👆 token will expire within the next 4 hours
});
```
{% endcode %}

{% code title="login.effect.ts" %}
```typescript
import { generateTokenPayload } from './token.helper';

const login$ = EffectFactory
  .matchPath('/login')
  .matchType('POST')
  .use(req$ => req$.pipe(
    map(req => req.body),
    mergeMap(UserDao.findByCredentials),
    map(generateTokenPayload),
    // 👇
    map(generateToken({ secret: Config.jwt.secret })),
    map(token => ({ body: { token } })),
    catchError(() => throwError(
      new HttpError('Unauthorized', HttpStatus.UNAUTHORIZED)
    )),
  ));
```
{% endcode %}

