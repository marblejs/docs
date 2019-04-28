# middleware-io

A data validation middleware based on awesome [io-ts](https://github.com/gcanti/io-ts) library authored by [_gcanti_](https://github.com/gcanti).

### Installation

```bash
$ npm i @marblejs/middleware-io
```

Requires `@marblejs/core` to be installed.

### Importing

```typescript
// HTTP
import { requestValidator$ } from '@marblejs/middleware-io';

// Events, eg. WebSockets
import { eventValidator$ } from '@marblejs/middleware-io';
```

### Type declaration <a id="type-declaration"></a>

```text
requestValidator$ :: (RequestSchema, ValidatorOptions) -> Observable<HttpRequest> -> Observable<HttpRequest>
eventValidator$ :: (Schema, ValidatorOptions) -> Observable<Event> -> Observable<Event>
```

### Parameters

_**requestValidator$**_

| parameter | definition |
| :--- | :--- |
| _schema_ | `Partial<RequestSchema>`\(see [io-ts](https://github.com/gcanti/io-ts) docs\) |
| _options_ | &lt;optional&gt; `ValidatorOptions` |

_**eventValidator$**_

| parameter | definition |
| :--- | :--- |
| _schema_ | `Schema` \(see [io-ts](https://github.com/gcanti/io-ts) docs\) |
| _options_ | &lt;optional&gt; `ValidatorOptions` |

_**ValidatorOptions**_

| parameter | definition |
| :--- | :--- |
| _reporter_ | &lt;optional&gt; `Reporter` |
| _context_ | &lt;optional&gt; `string` |

### Usage

Let's define a user schema that will be used for I/O validation.

{% code-tabs %}
{% code-tabs-item title="user.schema.ts" %}
```typescript
export const userSchema = t.type({
  id: t.string,
  firstName: t.string,
  lastName: t.string,
  roles: t.array(t.union([
    t.literal('ADMIN'),
    t.literal('GUEST'),
  ])),
});

export type User = t.TypeOf<typeof userSchema>;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```typescript
import { use, r } from '@marblejs/core';
import { requestValidator$, t } from '@marblejs/middleware-io';
import { userSchema } from './user.schema.ts';

const effect$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    use(requestValidator$({ body: userSchema })),
    // ..
  )));
```

{% hint style="info" %}
For more validation use cases and recipes, visit [Validation](../advanced/validation.md) chapter.
{% endhint %}

You can also reuse the same schema for Events validation if you want.

```typescript
import { matchEvent, use } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { eventValidator$, t } from '@marblejs/middleware-io';
import { userSchema } from './user.schema.ts';

const postUser$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('CREATE_USER'),
    use(eventValidator$(userSchema)),
    // ...
  );
```

The inferred `req.body` / `event.payload` type of provided schema, will be of the following form:

```typescript
type User = {
  id: string;
  firstName: string;
  lastName: string;
  roles: ('ADMIN' | 'GUEST')[];
};
```

### Validation errors

Lets take a look at the default reported validation error thrown by `eventValidator$` . Let's assume that client passed wrong values for `firstName` and `roles`  fields.

```javascript
payload.lastName = false;
payload.roles = ['TEST'];
```

The reported error intercepted via default error effect will look like follows.

```javascript
{
  type: 'CREATE_USER',
  error: {
    message: 'Validation error',
    data: [
      {
        path: 'lastName',
        expected: 'string',
        got: 'false'
      },
      {
        path: 'roles.0.0',
        got: '"TEST"',
        expected: '"ADMIN"'
      },
      {
        path: 'roles.0.1',
        got: '"TEST"',
        expected: '"GUEST"'
      }
    ]
  }
}
```

### Reporters

You can create custom reporters by conforming to [io-ts](https://github.com/gcanti/io-ts#error-reporters) `Reporter` interface.

```typescript
interface Reporter<A> {
  report: (validation: Validation<any>) => A
}
```

In order to use custom reporter you have to pass it with `options` object as a second argument.

```typescript
requestValidator$(schema, { reporter: customReporter });
```

