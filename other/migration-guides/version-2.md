---
description: >-
  This chapter provides a set of guidelines to help you migrate from Marble.js
  version 2.x to the latest 3.x version.
---

# Migration from version 2.x

The newest iteration comes with some API breaking change, but don’t worry, these are not game-changers, but rather convenient improvements that open the doors to new future possibilities. During the development process, the goal was to notify and discuss incoming changes within the community as early as possible. You can check here [what has changed since the latest version](https://github.com/marblejs/marble/issues/172).

## @marblejs/core

### fp-ts

`fp-ts@2.x` is a required peer dependency \(next to `rxjs`\)

{% page-ref page="../../getting-started/installation.md" %}

### Context API

`fp-ts@2.x` introduced changes that have a major impact to Context API \(eg. Reader monad\).

Version 3.0 introduces more explicit dependency binding. Previous API wasn't so precise, which could result to confusion, eg. when the dependency is lazily/eagerly evaluated.

{% page-ref page="../../http/context.md" %}

**❌ Old:**

```typescript
import { bindTo } from '@marblejs/core';

// eager
bindTo(WsServerToken)(websocketsServer.run),

// lazy
bindTo(WsServerToken)(websocketsServer),
```

**✅ New:**

```typescript
import { bindTo, bindLazilyTo, bindEagerlyTo } from '@marblejs/core';

// eager
bindEagerlyTo(WsServerToken)(websocketsServer),

// lazy
bindTo(WsServerToken)(websocketsServer),
bindLazilyTo(WsServerToken)(websocketsServer),
```

### Readers

**❌ Old:**

```typescript
import { reader } from '@marblejs/core';

const foo = reader.map(ctx => {
  // ...
});
```

**✅ New:**

You can create context readers via raw Reader monad composition \(using available fp-ts toolset\) or using `createReader` utility function that saves a lot of unnecessary boilerplate.

```typescript
import { createReader, reader } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import { map } from 'fp-ts/lib/Reader';

const foo1 = pipe(reader, map(ctx => {
  // ...
}));

// or much simpler

const foo2 = createReader(ctx => {
  // ...
});
```

### Server creators

The release of fp-ts also had an impact to HTTP and WebSocket server creators. `run()` method on Reader, etc. has been replaced with **IO** thunk. Additionally all server creators are promisified, which means that they will return an instance only when started listening. The change applies to all main modules: `@marblejs/core`, `@marblejs/websockets`, `@marblejs/messaging`. More info you can find in PR [\#198](https://github.com/marblejs/marble/pull/198)

Bootstrapping:

**❌ Old:**

```typescript
const server = createServer({
  // ...
});

server.run();
```

**✅ New:**

```typescript
const server = createServer({
  // ...
});

await (await server)();
```

Unified config interfaces for all kind of server creators:

{% tabs %}
{% tab title="websockets" %}
```typescript
import { createWebSocketServer, webSocketListener } from '@marblejs/websockets';

const webSocketServer = createWebSocketServer({
  // ...
  listener: webSocketListener({
    middlewares: [...],
    effects: [...],
  }),
});
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="http" %}
```typescript
import { createServer, httpListener } from '@marblejs/core';

const server = createServer({
  // ...
  listener: httpListener({
    middlewares: [...],
    effects: [...],
  }),
});
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="messaging" %}
```typescript
import { createMicroservice, messagingListener } from '@marblejs/messaging';

const microservice = createMicroservice({
  // ...
  listener: messagingListener({
    middlewares: [...],
    effects: [...],
  }),
});
```
{% endtab %}
{% endtabs %}

### **Effect**

Marble.js v2.0 `Effect` interface defines three arguments where the second one is used for accessing contextual client, eg. _HttpResponse_, _WebSocketClient_, etc. Typically the second argument was not used very often. That's why in the next major version client parameter moves to context object. It results in reduced available number of parameters from 3 to 2.

{% hint style="info" %}
The change applies to all Effect interfaces, eg. `HttpEffect`, `WsEffect`, `MsgEffect`
{% endhint %}

**❌ Old:**

```typescript
const foo$: WsEffect = (event$, client, meta) =>
  event$.pipe(
    matchEvent('FOO'),
    // meta.ask       ---    context reader
  );
```

**✅ New:**

```typescript
const foo$: WsEffect = (event$, ctx) =>
  event$.pipe(
    matchEvent('FOO'),
    // ctx.client    ---    contextual client
    // ctx.ask       ---    context reader
  );
```

This change also implies a much cleaner base `Effect` interface:

```typescript
interface Effect<I, O, Client> {
  (input$: Observable<I>, ctx: EffectContext<Client>): Observable<O>;
}

interface EffectContext<T, U extends SchedulerLike = SchedulerLike> {
  ask: ContextProvider;
  scheduler: U;
  client: T;
}
```

With that change the last argument of Effect interface is no more called as `EffectMetadata` but rather as `EffectContext`.

### ErrorEffect

When dealing with error or output Effect, the developer had to use the attribute placed in the third effect argument. In Marble.js v3.0 the thrown error is passed to stream directly.

**❌ Old:**

```typescript
const error$: HttpErrorEffect<HttpError> = (req$, client, { error }) =>
  req$.pipe(
    map(req => {
      // ...
    }),
  );
```

**✅ New:**

```typescript
const error$: HttpErrorEffect<HttpError> = req$ =>
  req$.pipe(
    map(({ req, error }) => {
      // ...
    }),
  );
```

### OutputEffect

**❌ Old:**

```typescript
const output$: HttpOutputEffect = (res$, client, { initiator }) =>
  res$.pipe(
    map(res => {
      // ...
    }),
  );
```

**✅ New:**

```typescript
const output$: HttpOutputEffect = res$ =>
  res$.pipe(
    map(({ req, res }) => {
      // ...
    }),
  );
```

### HttpRequest

The HttpResponse object is no more carried separately but together with correlated HttpRequest.

```typescript
interface HttpRequest {
  url: string;
  method: HttpMethod;
  body: Body;
  params: Params;
  query: Query;
  response: HttpResponse;     // 👈 
}
```

```typescript
const effect$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => req$.pipe(
    map(req => ...),
    // req.response -- HttpResponse object
    // ctx.client -- HttpServer object
  )));
```

## @marblejs/middleware-logger

The newest implementation of logger middleware uses underneath the pluggable Logger dependency, so dedicated log streaming is unnecessary. The developer can simply override default Logger binding and forward the incoming logs on his own. Also, since `HttpRequest` contains the response object attached to it - `res` attribute is redundant.

{% page-ref page="../../http/advanced/logging.md" %}

**❌ Old:**

```typescript
interface LoggerOptions {
  silent?: boolean;
  stream?: WritableLike;
  filter?: (res: HttpResponse, req: HttpRequest) => boolean;
}
```

**✅ New:**

```typescript
interface LoggerOptions {
  silent?: boolean;
  filter?: (req: HttpRequest) => boolean;
}
```

