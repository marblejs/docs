# Context

> From version 2.0, **Marble.js** takes even bigger steps in being a **functional** reactive framework. Besides the monadic flavour of Effect streams we try to incorporate more functional patterns and concepts which can fit well in the framework ecosystem.

## Dependency Injection

Dependency Injection \(DI\) is a very simple concept, which can be implemented in many different ways. It means to get dependencies of a class passed in by using constructor, or to get dependencies of a function passed in by using arguments, or even more advanced techniques. If we step back and look at the concept in a more abstract way, the only thing to remember is that we gain the possibility to provide dependencies to any of our entities any point in time. Now we can provide different implementations of those dependencies by using extension \(polymorphism\), interface implementation, or whatever technique we want to use.

Marble.js comes to the DI concept in a different, more functional way, that can be very similar to popular pure functional languages like eg. Haskell. From version 2.0, Marble.js introduces a **Context**, which is an abstraction over Reader monad implementation of the DI system.

What [Haskell docs](http://hackage.haskell.org/package/mtl-2.2.2/docs/Control-Monad-Reader.html) says about the Reader monad?

> The [`Reader`](http://hackage.haskell.org/package/mtl-2.2.2/docs/Control-Monad-Reader.html#t:Reader) monad \(also called the Environment monad\), represents a computation, which can read values from a shared environment, pass values from function to function, and execute sub-computations in a modified environment. \[...\]

## The basics

In Marble.js you don't have to create the app context explicitly. In order to create a basic environment you can use `createServer` function which prepares underneath a basic application context.

Every dependency that you would like to register inside the Context has to conform to `ContextReader` interface, which means that the registered function should be able to read from the bootstrapped server context. Knowing the basics, let's create some readers!

{% code-tabs %}
{% code-tabs-item title="example.ts" %}
```typescript
import { createContextToken, reader } from '@marblejs/core';

export const d1Token = createContextToken<string>();
export const d2Token = createContextToken<string>();

export const d1 = reader.map(() => 'Hello');
export const d2 = reader.map(ask =>
  ask(d1Token).map(v => v + ', world!').getOrElse('')
);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="index.ts" %}
```typescript
import { bindTo createServer } from '@marblejs/core';
import { d1, d2, d1Token, d2Token } from './example';

createServer({
  // ...
  dependencies: [
    bindTo(d1Token)(d1),
    bindTo(d2Token)(d2),
  ],
  // ...
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Having our dependencies defined, let's define some test Effect where we can test how our dependency can be consumed. 

{% code-tabs %}
{% code-tabs-item title="example.effect.ts" %}
```typescript
import { r } from '@marblejs/core';
import { d2Token } from './example';

export const example$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, _, { ask }) => req$.pipe(
    mapTo(ask(d2Token).getOrElse('')),
    map(msg => ({ body: msg })),
  )));
```
{% endcode-tabs-item %}
{% endcode-tabs %}

The type safety is very important. If you are percipiet, you'll notice that using previously defined `d2Token` together with provided dependency we can also grab its inferred type. Reading from the context is not safe every time, thats why the provided dependency is wrapped arround [`Option`](https://gcanti.github.io/fp-ts/Option.html) monad that you can work on. As you can see the real benefit of using Readers is to be able to provide that context in an implicit way without the need to state it explicitly on each one of the functions that needs it.

If you will try to do a `GET /` request, you should see in the `Hello, world!` message in the respone. Thats how Dependency Injection work in Marble.js!

## Eager vs lazy readers

Let's say you have a HTTP server that would like to connect with a WebSocket server. When bootstrapping a WebSocket server we want to instantiate it as soon as possible \(aka eagerly\). The Marble.js Context was designed with a need for flexible way of connecting dependent modules - eagerly and lazily.

**By default Instances are created lazily when they are needed**. If a dependency is never used by another component, then it won’t be created at all. This is usually what you want. For most components there’s no point creating them until they’re needed. However, in some cases you want dependencies to be started up straight away or even if they’re not used by another function. For example, you might want to send a message to a remote system, warm up a cache when the application starts or boostrapp a WebSocket server. You can force a dependency to be created eagerly by using an **eager binding**.

Lets look at an example of eagerly binding of a WebSocket server.

{% code-tabs %}
{% code-tabs-item title="tokens.ts" %}
```typescript
import { createContextToken } from '@marblejs/core';
import { MarbleWebSocketServer } from '@marblejs/websockets';

export const WsServerToken = createContextToken<MarbleWebSocketServer>();
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="index.ts" %}
```typescript
import { createServer, bindTo } from '@marblejs/core';
import { mapToServer } from '@marblejs/websockets';
import { WsServerToken } from './tokens';
import httpListener from './http.listener';
import webSocketListener from './ws.listener';

const server = createServer({
  port: 1337,
  httpListener,
  dependencies: [
    bindTo(WsServerToken)(webSocketListener({ port: 8080 }).run),
  ],
});

server.run();
```
{% endcode-tabs-item %}
{% endcode-tabs %}

In order to instantiate our registered dependency as soon as possible, you have to run it inside `bindTo` function. It means that the registered dependency will try to resolve its dependencies during the binding, using previously registered context.

{% hint style="warning" %}
Note that in order to run registered dependency eagerly you have the provide proper context by registering dependent component before the eager dependency.
{% endhint %}

```typescript
// lazy binding
bindTo(Token)(dependency());

// eager binding
bindTo(Token)(dependency().run);
```

Having the WebSocket dependency eagerly registered we can ask for it eg. inside HTTP Effect. Note that provided dependency won't be instantiated while asking - we can easily grab previously instantiated WebSocket server on demand. 😎

{% code-tabs %}
{% code-tabs-item title="http.listener.ts" %}
```typescript
import { httpListener } from '@marblejs/core';
import { requestValidator$ } from '@marblejs/middleware-io';
import { bodyParser$ } from '@marblejs/middleware-body';
import { WsServerToken } from './tokens';

const postItem$ = r.pipe(
  r.matchPath('/items'),
  r.matchType('POST'),
  r.useEffect((req$, _, { ask }) => req$.pipe(
    use(requestValidator$({ body: itemDto }),
    // ...
    tap(item => ask(WsServerToken).map(server =>
      server.sendBroadcastResponse({ type: 'ADDED_ITEM', payload: item })),
    ),
    map(item => ({ body: item })),
  )));

export default httpListener({
  middlewares: [bodyParser$()],
  effects: [postItem$]
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

