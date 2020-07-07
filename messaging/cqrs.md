# CQRS

{% hint style="warning" %}
This chapter doesn't intend to describe the idea that stands behind CQS/CQRS. If you would like to learn more about it, there are many good resources that explain the topic in a better way.
{% endhint %}

## Installation

```bash
$ yarn add @marblejs/core @marblejs/messaging rxjs fp-ts
```

## Overview

The design has as its main concern, the separation of read and write operations The pattern divides methods into three categories: commands, queries and events.

{% hint style="success" %}
Marble.js implements a dedicated “local” transport layer for EventBus messaging, which can be easily adopted to CQS/CQRS pattern. Compared to other popular implementations, the module implements only one, common bus for transporting events of any type.
{% endhint %}

#### Commands

Dispatching a command is akin to invoking a method: we want to do something specific. This means we can only have one place, one handler, for every command. This also means that we may expect a reply: either a value or an error. Firing command is the only way to change the state of our system - they are responsible for introducing all changes to the system.

```typescript
{
  type: 'CREATE_USER',
  payload: {
    firstName: 'John',
    lastName: 'Doe',
  }
}
```

#### Queries

Queries never modify the database - they only perform read operations so they don't affect the state of the system. A query returns a DTO that does not encapsulate any domain knowledge.

```typescript
{
  type: 'GET_USER_BY_ID',
  payload: {
    id: '#ABC123',
  }
}
```

#### Events

When dispatching an event, we notify the application about the change. We do not get a reply and there might be more than one effect handler registered.

```typescript
{
  type: 'USER_CREATED',
  payload: {
    id: '#ABC123',
  }
}
```

### EventBus

In order to initialize event bus you have bind the dependency to the server. Since the context resolving can occur in an asynchronous way the dependency has to be bound eagerly \(on app startup\). The factory inherits all bounded dependencies from the server, so there is not need to register the same dependencies one more time.

```typescript
import { bindEagerlyTo } from '@marblejs/core';
import { EventBusToken, eventBus } from '@marblejs/messaging';

const listener = messagingListener({
  middlewares: [ ... ],
  effects: [ ... ],
});

// ...

bindEagerlyTo(EventBusToken)(eventBus({ listener })),
```

### EventBus Client

EventBus client is a specialized form of messaging client that operates on local transport layer. It can be injected into any effect via `EventBusClientToken`. Due to its async nature it has to be bound eagerly on app startup.

```typescript
import { bindEagerlyTo } from '@marblejs/core';
import { EventBusClientToken, eventBusClient } from '@marblejs/messaging';

// ...

bindEagerlyTo(EventBusClientToken)(eventBusClient),
```

Similar to every messaging client, the reader exposes two main methods:

| method | description |
| :--- | :--- |
| `send` | Blocking \(**request-response**\), RPC-like messaging. |
| `emit` | Asynchronous messaging without response. |
| `close` | Closes event bus subscribers. |

## Example

Let's build a simple event-based app that demonstrates how to dispatch commands from an endpoint and process them in a command handler.

{% tabs %}
{% tab title="user.command.ts" %}
```typescript
import { event } from '@marblejs/core';
import * as t from 'io-ts';

export enum UserCommandType {
  CREATE_USER = 'CREATE_USER',
};

export const CreateUserCommand =
  event(UserCommandType.CREATE_USER)(t.type({
    firstName: t.string,
    lastName: t.string,
  }));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="user.event.ts" %}
```typescript
import { event } from '@marblejs/core';
import * as t from 'io-ts';

export enum UserEventType {
  USER_CREATED = 'USER_CREATED',
};

export const UserCreatedEvent =
  event(UserCommandType.USER_CREATED)(t.type({
    id: t.string,
  }));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="createUser.effect.ts" %}
```typescript
import { act, useContext, matchEvent } from '@marblejs/core';
import { reply, MsgEffect } from '@marblejs/messaging';
import { mergeMap } from 'rxjs/operators';
import { pipe } 'fp-ts/lib/pipeable';
import { createUser } from './user.model';
import { CreateUserCommand } from './user.command';
import { UserCreatedEvent } from './user.event';
import { UserRespositoryToken } from './tokens';

export const createUser$: MsgEffect = (event$, ctx) => {
  const userRepository = useContext(UserRespositoryToken)(ctx.ask);
  
  return event$.pipe(
    matchEvent(CreateUserCommand),
    act(event => pipe(
      event.payload,
      createUser,
      userRepository.persist,
      mergeMap(user => [
        UserCreatedEvent.create({ id: user.id }),
        reply(event)(UserCreatedEvent.create({ id: user.id })),
      ]),
    )),
  );
};
```
{% endtab %}
{% endtabs %}

The logic of the command handler is quite simple:

1. match all incoming events \(commands\) of specific type \(`CREATE_USER`\)
2. create domain object and persist via injected repository
3. notify all interested parties that user was created
4. return a confirmation back to client

{% tabs %}
{% tab title="postUser.effect.ts" %}
```typescript
import { r, HttpStatus, useContext, use } from '@marblejs/core';
import { map, mapTo, mergeMap } from 'rxjs/operators';
import { EventBusClientToken } from '@marblejs/messaging';
import { requestValidator$, t } from '@marblejs/middleware-io';
import { CreateUserCommand } from './user.commands';
import { pipe } from 'fp-ts/lib/pipeable';

const validateRequest = requestValidator$({
  body: t.type({
    firstName: t.string,
    lastName: t.string,
  }),
});

export const postUser$ = r.pipe(
  r.matchPath('/user'),
  r.matchType('POST'),
  r.useEffect((req$, ctx) => {
    const eventBusClient = useContext(EventBusClientToken)(ctx.ask);

    return req$.pipe(
      validateRequest,
      mergeMap(req => {
        const { firstName, lastName } = req.body;
        
        return pipe(
          CreateUserCommand.create({ firstName, lastName }),
          eventBusClient.send,
        );
      }),
      mapTo({ status: HttpStatus.CREATED }),
    );
  }));
```
{% endtab %}
{% endtabs %}

The implementation of `postUser$` effect is also very simple. First we have to inject the EventBus client from the context and map the incoming request to `CREATE_USER` command. Since we want to notify the API consumer about success or failure of operation, we have to wait for the response. In order to do that we have to dispatch an event using `send` method.

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { httpListener, createServer, bindEagerlyTo } from '@marblejs/core';
import { messagingListener, EventBusToken, EventBusClientToken, eventBusClient, eventBus } from '@marblejs/messaging';
import { bodyParser$ } from '@marblejs/middleware-body';
import { logger$ } from '@marblejs/middleware-logger';
import { postUser$ } from './postUser.effect';
import { createUser$ } from './createUser.effect';

const eventBusListener = messagingListener({
  effects: [
    createUser$,
  ],
});

const listener = httpListener({
  middlewares: [
    logger$(),
    bodyParser$(),
  ],
  effects: [
    postUser$,
  ],
});

export const server = createServer({
  listener,
  dependencies: [
    bindEagerlyTo(EventBusClientToken)(eventBusClient),
    bindEagerlyTo(EventBusToken)(eventBus({ listener: eventBusListener })),
  ],
});

const main: IO<void> = async () =>
  await (await server)();

main();
```
{% endtab %}
{% endtabs %}

