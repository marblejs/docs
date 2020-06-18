---
description: >-
  DI is a very simple concept, which can be implemented in many different ways.
  Marble.js introduces a Context, which is an abstraction over Reader monad
  implementation of the DI system.
---

# Context

## Dependency Injection

Dependency Injection \(DI\) is a very simple concept, which can be implemented in many different ways. It means to get dependencies of a class passed in by using constructor, or to get dependencies of a function passed in by using arguments, or even more advanced techniques. If we step back and look at the concept in a more abstract way, the only thing to remember is that we gain the possibility to provide dependencies to any of our entities any point in time. Now we can provide different implementations of those dependencies by using extension \(polymorphism\), interface implementation, or whatever technique we want to use.

Marble.js comes to the DI concept in a different, more functional way, that can be very similar to popular pure functional languages like eg. Haskell. From version 2.0, Marble.js introduces a **Context**, which is an abstraction over Reader monad implementation of the DI system.

> The [`Reader`](http://hackage.haskell.org/package/mtl-2.2.2/docs/Control-Monad-Reader.html#t:Reader) monad \(also called the Environment monad\), represents a computation, which can read values from a shared environment, pass values from function to function, and execute sub-computations in a modified environment. \[...\]
>
> ~ [Haskell docs](http://hackage.haskell.org/package/mtl-2.2.2/docs/Control-Monad-Reader.html)

## The basics

In Marble.js you don't have to create the app context explicitly. In order to create a basic environment you can use `createServer` function which prepares underneath a basic application context with default set of bounded dependencies, like, eg. [_Logger_](advanced/logging.md).

Every context dependency that you would like to register has to conform to `ContextReader` interface, which in other words means that the registered function should be able to read from the bootstrapped server context. Knowing the basics, let's create some readers!

{% tabs %}
{% tab title="example.ts" %}
```typescript
import { createContextToken, reader } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import * as R from 'fp-ts/lib/Reader';
import * as O from 'fp-ts/lib/Option';

export const Dependency1Token = createContextToken<string>('Dependency1');
export const Dependency2Token = createContextToken<string>('Dependency2');

export const Dependency1 = pipe(reader, R.map(() => 'Hello'));
export const Dependency2 = pipe(reader, R.map(ask => pipe(
  ask(Dependency1Token),
  O.map(v => v + ', world!'),
  O.getOrElse(() => ''),
)));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { bindTo createServer } from '@marblejs/core';
import { Dependency1, Dependency2, Dependency1Token, Dependency2Token } from './example';

const server = createServer({
  // ...
  dependencies: [
    bindTo(Dependency1Token)(Dependency1),
    bindTo(Dependency2Token)(Dependency2),
  ],
  // ...
});
```
{% endtab %}
{% endtabs %}

Having our dependencies defined, let's define some test Effect where we can check how our dependency can be consumed.

{% tabs %}
{% tab title="example.effect.ts" %}
```typescript
import { r } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import * as O from 'fp-ts/lib/Option';
import { Dependency2Token } from './example';

export const example$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => {
  
    const dependency2 = pipe(
      ctx.ask(Dependency2Token),
      O.getOrElse(() => ''),
    );
    
    return req$.pipe(
      mapTo(dependency2),
      map(body => ({ body })),
    ));
  });
```
{% endtab %}
{% endtabs %}

If you will try to do a `GET /` request, you should see in the `Hello, world!` message in the response. Thats how Dependency Injection work in Marble.js!

Each Marble.js Effect defines a second argument called as `EffectContext` which holds i.a. the context provider \(**ask**\) and the contextual client instance \(in case of HTTP module it will be a running _HttpServer_\).

The type safety is very important. If you are percipient, you'll notice that by using previously defined `Dependency2Token` , we can also grab the dependency inferred type. Reading from the context is not a safe operation, thus the provided dependency is wrapped around [Option](https://gcanti.github.io/fp-ts/Option.html) monad that you can work on. As you can see the real benefit of using Readers is to be able to provide that context in an implicit way without the need to state it explicitly on each one of the functions that needs it.

{% hint style="info" %}
All Marble.js Effects are eagerly bootstrapped, which means that we you can inject dependencies only once at app startup, if the dependency is injected before the main _Observable_ stream.
{% endhint %}

## createReader + useContext

As you can see reading from context is a very verbose operation - you have to pipe the reader instance, ask the context provider with a token, map the result and add a fallback in case of unmeet dependency. That's a lot of work to do! What is really needed is to resolve all required dependencies before the startup by asking the context and failing in case of unmeet dependency - that's the typical use case. Going out towards expectations, Marble.js defines a useful `createReader` and `useContext` utility functions that save a lot of unnecessary boilerplate. Let's redefine the previous example.

```typescript
import { createContextToken, createReader } from '@marblejs/core';

export const Dependency1Token = createContextToken<string>('Dependency1');
export const Dependency2Token = createContextToken<string>('Dependency2');

export const Dependency1 = createReader(() => 'Hello');
export const Dependency2 = createReader(ask =>
  useContext(Dependency1Token)(ask) + ', world!');
```

```typescript
import { r, useContext } from '@marblejs/core';
import { Dependency2Token } from './example';

export const example$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => {
  
    const dependency2 = useContext(Dependency2Token)(ctx.ask);
    
    return req$.pipe(
      mapTo(Dependency2),
      map(body => ({ body })),
    ));
  });
```

{% hint style="info" %}
In order to have a more grained control over injected context dependencies, please use a raw Reader monad.
{% endhint %}

## Eager vs lazy readers

Let's say you have a HTTP server that would like to connect with another one. When bootstrapping a WebSocket server we want to instantiate it as soon as possible \(aka eagerly\). The Marble.js Context was designed with a need for flexible way of connecting dependent modules - eagerly or lazily.

**By default Instances are created lazily - when they are needed**. If a dependency is never used by another component, then it won’t be created at all. This is usually what you want. For most readers there’s no point creating them until they’re needed. However, in some cases you want your dependencies to be started up straight away or even if they’re not used by another function. For example, you might want to send a message to a remote system or warm up a cache when the application starts. You can force a dependency to be created eagerly by using an **eager binding**.

In order to instantiate our registered dependency as soon as possible, you have to run it inside `bindEagerlyTo` function. It means that the registered dependency will try to resolve its dependencies on server startup.

{% hint style="warning" %}
Note that the order of registered lazy dependencies doesn't matter. Marble.js will start to resolve eager dependencies on on app startup, when all dependencies are already bound.
{% endhint %}

```typescript
import { bindTo, bindLazilyTo, bindEagerly } from '@marblejs/core';

// lazy binding
bindTo(Token)(Dependency);
bindLazilyTo(Token)(Dependency);

// eager binding
bindEagerly(Token)(Dependency);
```

## Async readers

Sometimes there is a need to suspend the application startup until one or more asynchronous tasks or jobs are fulfilled. For example, you may want to wait with starting up your server before the connection with a database has been established. The updated syntax of context readers handles Promises or async/await syntax out of the box in the reader factory. The context container \(including Marble app factory\) will await a resolution of the promise before instantiating any reader that depends on \(injects\) async reader.

```typescript
import { bindTo, bindEagerly } from '@marblejs/core';

bindEagerlyTo(Token)(async () => 'bar');

const foo = useContext(Token)(ask);   // foo === 'bar'

// but...

bindTo(Token)(async () => 'bar');

const foo = useContext(Token)(ask);   // foo === Promise<'bar'>
```

Let's look at an example of eager binding of a WebSocket server.

{% tabs %}
{% tab title="tokens.ts" %}
```typescript
import { createContextToken } from '@marblejs/core';
import { WebSocketServerConnection } from '@marblejs/websockets';

export const WebSocketServerToken =
  createContextToken<WebSocketServerConnection>('WebSocketServer');
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { createServer, bindEageryTo } from '@marblejs/core';
import { mapToServer } from '@marblejs/websockets';
import { WebSocketServerToken } from './tokens';
import { webSocketServer } from './websocket.server';

const server = createServer({
  // ,.
  dependencies: [
    bindEageryTo(WebSocketServerToken)(async () =>
      await (await webSocketServer)()
    ),
  ],
});
```
{% endtab %}
{% endtabs %}

Having the WebSocket dependency eagerly registered we can ask for it inside an Effect.

{% hint style="info" %}
Note that provided dependency won't be instantiated one more time while asking since it is already evaluated.
{% endhint %}

{% tabs %}
{% tab title="postItem.effect.ts" %}
```typescript
import { useContext } from '@marblejs/core';
import { requestValidator$, t } from '@marblejs/middleware-io';
import { bodyParser$ } from '@marblejs/middleware-body';
import { map, mergeMap } from 'rxjs/operators';
import { WebSocketServerToken } from './tokens';

const validator$ = requestValidator$({
  body: t.type(...),
});

const postItem$ = r.pipe(
  r.matchPath('/items'),
  r.matchType('POST'),
  r.useEffect((req$, ctx) => {
    const webSocketServer = useContext(WebSocketServerToken)(ctx.ask);
    
    return req$.pipe(
      use(validator$),
      map(req => req.body),
      mergeMap(payload =>
        webSocketServer.sendBroadcastResponse({ type: 'ADDED_ITEM', payload })),
      // ...
      map(body => ({ body })),
    ));
  });
```
{% endtab %}
{% endtabs %}

