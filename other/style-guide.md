# Style Guide

{% hint style="info" %}
The style guide is mostly based on author experience in using Marble.js on large production-ready projects following microservice/event-driven architecture style.
{% endhint %}

## Project organization

Marble.js framework doesn't define any strict file structures and conventions from the organization-level perspective that each developer should enforce. Below you can find some useful hints that you can follow when organizing your Marble.js app.

### Keep server and connected listeners in separate files

From the framework perspective, **listeners** and **servers** are separate layers that are responsible for different things. [\#createServer](api-reference/core/createserver.md) and similar factory functions are responsible for handling transport-layer-related processes, like: bootstrapping server and bounded context, listening for incoming messages or reacting for transport-layer events, where for the contrast, listeners are responsible for processing I/O messages that go through underlying transport layer in order to fulfill business needs. Basically, it is just about the single responsibility principle.

By convention Marble.js follows suffixed file naming which results in:

| Server | Listener |
| :--- | :--- |
| `http.server.ts` | `http.listener.ts` |
| `microservice.server.ts` | `microservice.listener.ts` |
| `websocket.server.ts` | `websocket.listener.ts` |
| `NONE` | `eventbus.listener.ts` |

### Keep HTTP route with its corresponding HttpEffect

In Marble.js HTTP effects are tightly connected to the route on which they operate. Having them in a separation makes the code more less readable and understandable. Always try to keep them as a one unit. Every HttpEffect that comes through `r.pipe` route builder is accessible via `.effect` property of the returned `RouteEffect` object. You can access it via this way if you want to unit test only the effect function.

**❌ Bad**

{% tabs %}
{% tab title="getFoo.route.ts" %}
```typescript
import { r } from '@marblejs/core';
import { getFooEffect } from './getFoo.effect';

const getFoo = r.pipe(
  r.matchPath('/foo'),
  r.matchType('GET'),
  r.useEffect(getFooEffect));
```
{% endtab %}

{% tab title="getFoo.effect.ts" %}
```typescript
import { HttpEffect } from '@marblejs/core';

export const getFooEffect: HttpEffect = (req$, ctx) => req$.pipe(
  ...
);
```
{% endtab %}
{% endtabs %}

✅ **Good**

{% tabs %}
{% tab title="getFoo.effect.ts" %}
```typescript
import { r } from '@marblejs/core';

const getFoo$ = r.pipe(
  r.matchPath('/foo'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => req$.pipe(
    ...
  )));
```
{% endtab %}
{% endtabs %}

### Use index files for combining related effects

Index files are good for combining and re-exporting related files as a one module.

```text
/user
  + ── /getUserList
  │      + ── getUserList.effect.spec.ts
  │      └─── getUserList.effect.ts
  │
  + ── /getUser
  │      + ── getUser.effect.spec.ts
  │      └─── getUser.effect.ts
  │
  + ── /postUser
  │      + ── getUser.effect.spec.ts
  │      └─── getUser.effect.ts
  │
  └─── index.ts
  
```

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { combineRoutes } from '@marblejs/core';

export const user$ = combineRoutes('/user', {
  middlewares: [
    authorize$,
  ],
  effects: [
    getUserList$,
    getUser$,
    postUser$,
  ],
});
```
{% endtab %}

{% tab title="http.listener.ts" %}
```typescript
import { httpListener } from '@marblejs/core';
import { user$ } from './user';

export const listener = httpListener({
  effects: [
    user$,
  ],
});
```
{% endtab %}
{% endtabs %}

Or in case basic effects:

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { combineEffects } from '@marblejs/core';

export const user$ = combineEffects(
  userCreated$,
  userRemoved$,
  ...
);
```
{% endtab %}

{% tab title="eventBus.listener.ts" %}
```typescript
import { messagingListener } from '@marblejs/messaging';
import { user$ } from './user';

export const listener = messagingListener({
  effects: [
    user$,
  ],
});
```
{% endtab %}
{% endtabs %}

## Context

### Token naming and creation

* Use PascalCase naming convention with `Token` suffix for token definitions.
* Always remember to define a context token name identifier. It will help you in quick recognizing what dependency is missing when asking for a dependency via `useContext` hook function.
* Place token next to reader definition. It will be easier to navigate to reader implementation via popular "Go to Implementation" mechanism.

❌ **Bad**

{% tabs %}
{% tab title="tokens.ts" %}
```typescript
import { createContextToken } from '@marblejs/core';

export const userRepository = createContextToken<UserRepository>();
```
{% endtab %}

{% tab title="user.repository.ts" %}
```typescript
import { createReader } from '@marblejs/core';

export const userRepository = createReader(() => ...);
```
{% endtab %}
{% endtabs %}

✅ **Good**

{% tabs %}
{% tab title="user.repository" %}
```typescript
import { createContextToken, createReader } from '@marblejs/core';

export type UserRepository = ReturnType<typeof UserRepository>;

export const UserRepository = createReader(() => ...);

export const UserRepositoryToken = createContextToken<UserRepository>('UserRepository');
```
{% endtab %}
{% endtabs %}

### Eager vs Lazy binding

Identify which dependencies should be bound to the app context eagerly \(before app startup\) and which lazily.

**Given:**

```typescript
import { createContextToken, createReader } from '@marblejs/core';

export type DatabaseConnection = Connection;

export const DatabaseConnection = createReader(async _ => ...);

export const DatabaseConnectionToken = createContextToken<DatabaseConnection>('DatabaseConnection');
```

Note that in case of [async readers](../http/context.md#async-readers) the type of created reader will be: `Reader<Context, Promise<Connection>>`

❌ **Bad**

```typescript
import { bindTo, bindLazilyTo, useContext } from '@marblejs/core';

const dependencies = [
  bindTo(DatabaseConnectionToken)(DatabaseConnection),
  // or 
  bindLazilyTo(DatabaseConnectionToken)(DatabaseConnection),  
];

...

const connection = useContext(DatabaseConnectionToken)(ask);

// typeof connection === Promise<Connection>;
```

✅ **Good**

```typescript
import { bindEagerlyTo, useContext } from '@marblejs/core';

const dependencies = [
  bindEagerlyTo(DatabaseConnectionToken)(DatabaseConnection),
];

...

const connection = useContext(DatabaseConnectionToken)(ask);

// typeof connection === Connection;
```

### Injection

Always try to inject bound dependencies at the top level of your effects \(before returned Observable stream\). All effects are evaluated eagerly, so in case of missing context dependency the framework will be able to spot issues during initial bootstrap. 

❌ **Bad**

```typescript
const postUser$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect((req$, ask) => req$.pipe(
    use(postUserValidator$),
    mergeMap(req => {
      const userRepository = useContext(UserRepositoryToken)(ask);
      const { body } = req;
      
      return userRepository
        .persist(body)
        .pipe(
          mergeMap(userRepository.getById),
          map(user => ({ body: user })),
        );
    }),
  ));
```

✅ **Good**

```typescript
const postUser$ = r.pipe(
  r.matchPath('/'),
  r.matchType('POST'),
  r.useEffect((req$, ask) => {
    const userRepository = useContext(UserRepositoryToken)(ask);

    return req$.pipe(
      use(postUserValidator$),
      map(req => req.body),
      mergeMap(userRepository.persist),
      mergeMap(userRepository.getById),
      map(user => ({ body: user })),
    );
  }));
```

## Messaging

### Event encoding/decoding

* Use **UPPER\_SNAKE\_CASE** event type naming
* Use enumerable string literal type or plain const record for gathering a map of all possible event types for given context
* Use [`event`](../messaging/core-concepts/events.md#i-o-event-decoding-encoding) builder for I/O encoding/decoding
* Separate group your messages into `Events`, `Commands` or `Queries` \(see: [CQRS](../messaging/cqrs.md) chapter\)

❌ **Bad**

{% tabs %}
{% tab title="user.event.ts" %}
```typescript
import { Event } from '@marblejs/core';
import * as t from 'io-ts';

export const UserCreated = (id: string): Event => ({
  type: 'UserCreated',
  payload: { id },
});

export const UserUpdated = (id: string): Event => ({
  type: 'UserUpdated',
  payload: { id },
});

export const UserCreatedCodec = t.type({
  type: t.literal('UserCreated'),
  payload: t.type({ id: t.string }),
});

export const UserUpdatedCodec = t.type({
  type: t.literal('UserUpdated'),
  payload: t.type({ id: t.string }),
});
```
{% endtab %}
{% endtabs %}

**✅ Good**

```typescript
import { event } from '@marblejs/core';
import * as t from 'io-ts';

export enum UserEventType {
  USER_CREATED = 'USER_CREATED',
  USER_UPDATED = 'USER_UPDATED',
}

export const UserCreatedEvent =
  event(UserEventType.USER_CREATED)(t.type({
    id: t.string,
  }));
  
export const UserUpdatedEvent =
  event(UserEventType.USER_UPDATED)(t.type({
    id: t.string,
  }));
```

### Event matching and validation

* Always try match events by event I/O codec - avoid raw literals since they don't carry the actual event payload type
* Event-based communication follows the same laws as request-based communication - each incoming event should be validated before usage \(eg. using previously mentioned `event` codec\).

❌ **Bad**

{% tabs %}
{% tab title="userCreated.effect.ts" %}
```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';

export const userCreated$: MsgEffect = event$ =>
  event$.pipe(
    // event payload is unknown, no type is inferred... 
    matchEvent('USER_CREATED'),
    act(event => ...),
  );
```
{% endtab %}
{% endtabs %}

❌ / **✅ Better**

{% tabs %}
{% tab title="userCreated.effect.ts" %}
```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { UserCreatedEvent } from './user.event.ts';

export const userCreated$: MsgEffect = event$ =>
  event$.pipe(
    // type is inferred but event still requires validation...
    matchEvent(UserCreatedEvent),
    act(event => ...),
  );
```
{% endtab %}
{% endtabs %}

**✅ Good**

{% tabs %}
{% tab title="userCreated.effect.ts" %}
```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { UserCreatedEvent } from './user.event.ts';

export const userCreated$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent(UserCreatedEvent),
    act(eventValidator$(UserCreatedEvent)),
    act(event => ...),
  );
```
{% endtab %}
{% endtabs %}

### Effect output

* Each processed event should be mapped to a different event type to avoid infinite-loops.
* In case you don't want to emit anything in the effect stream you can skip emitted values, eg. by `ignoreElements` operator.

❌ **Bad**

```typescript
import { matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    tap(doSomeWork),
  );
```

**✅ Good**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { pipe } from 'fp-ts/lib/pipeable';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => pipe(
      doSomeWork(event),
      map(payload => ({ type: 'FOO_RESULT', payload })),
    )),
  );
```

### Error handling

* Each messaging effect should handle errors in a disposable streams either via `mergeMap/switchMap/...` operators with combination of `catchError` or via`act` operator.

❌ **Bad**

```typescript
import { matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    mergeMap(doSomeWork),
    map(payload => ({ type: 'FOO_RESULT', payload }),
  );
```

✅ **Good**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { pipe } from 'fp-ts/lib/pipeable';
import { catchError } from 'rxjs/operators';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    mergeMap(event => pipe(
      doSomeWork(event),
      map(payload => ({ type: 'FOO_RESULT', payload })),
      catchError(error => ({ type: 'FOO_ERROR', error })),
    )),
  );
```

**✅ Even better**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { pipe } from 'fp-ts/lib/pipeable';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => pipe(
      doSomeWork(event),
      map(payload => ({ type: 'FOO_RESULT', payload })),
    )),
  );
```

