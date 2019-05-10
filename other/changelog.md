# Changelog

## 2.1.1 - 2019-04-10

**Whats new?**

* **@marblejs/core**: added more HTTP statuses \#123 \(closes: \#121\)
* **@marblejs/core**: ability to grab the initial request from HTTP `output$` via `EffectMetadata.initiator` \#124 \(closes: \#122\)

```typescript
const output$: HttpOutputEffect = (res$, _, { initiator }) =>
  res$.pipe(
    // do something with response
  );
```

**Fixes**

* **@marblejs/core**: `response.handler` improved the way of how Node.js streams are detected \#125 

## 2.1.0 - 2019-04-02

**Whats new?**

* **@marblejs/core**: added support for passing Node.js streams into **HttpEffectResponse** `body` \#116 \(closes: \#114\)
* **@marblejs/middleware-cors**: introducing the new middleware that can be used to enable CORS with various options \#100 \(author: @Edouardbozon\) 👏🎉
* **@marblejs/middleware-logger**: `filter` can handle incoming requests and outgoing responses \#117 \(closes: \#115\) 

## 2.0.1 - 2019-03-14

**Whats new?**

* **@marblejs/core** - added missing server events parameters

## 2.0.0 - 2019-02-02

Over the past few months, we've been working hard on the next generation of Marble.js. Today we're thrilled to announce the second major release and all the exciting features that come with it. 🔥🎉🚀

![](https://media.giphy.com/media/3oKIPu1AxMWB2xlwl2/giphy.gif)

## 2.0.0-rc.3 - 2019-02-24

`pre-release`

**Whats new?**

* **@marblejs/core** - replaced `StaticInjectorContainer` with `Context` API [\#106 ](https://github.com/marblejs/marble/pull/106)
* **@marblejs/core** - running HTTP server is not registered anymore in context API [\#106 ](https://github.com/marblejs/marble/pull/106)
* **@marblejs/core** - `createServer` doesn't start listening automatically. You have to run it manually via `createServer().run()`**;** [\#106](https://github.com/marblejs/marble/pull/106)
* **@marblejs/core** - replaced `bind.to` function with curried `bindTo` [\#106 ](https://github.com/marblejs/marble/pull/106)
* **@marblejs/websockets** - module doesn't depend anymore on running HTTP server dependency [\#106](https://github.com/marblejs/marble/pull/106) 

**Breaking changes**

In order to use `httpListener` directly connected to `http.createServer` you have to run Reader context first:

```typescript
import { createContext } from '@marblejs/core';
import * as http from 'http';
import httpListener from './http.listener';

const httpListenerWithContext = httpListener
  .run(createContext());

export const server = http
  .createServer(httpListenerWithContext)
  .listen(1337, '127.0.0.1');
```

## 2.0.0-rc.2 - 2019-02-17

`pre-release`

**Whats new?**

* **@marblejs/core** - added new API for defining HTTP route via monadic pipe builder [\#103](https://github.com/marblejs/marble/pull/103)
* **@marblejs/core** - added support for HTTPS servers [\#105](https://github.com/marblejs/marble/pull/105)
* **@marblejs/core** -  fixed incorrectly parsed query parameters [\#104 ](https://github.com/marblejs/marble/pull/104)
* **@marblejs/middleware-body** - fixed incorrectly parsed URL-encoded parameters [\#104 ](https://github.com/marblejs/marble/pull/104)

## 2.0.0-rc.1 - 2019-02-08

`pre-release`

**Whats new?**

* **@marblejs/middleware-body** - Extended middleware API by customizable parsers [\#102 ](https://github.com/marblejs/marble/pull/102)
* **@marblejs/middleware-body** - The middleware can pass the request through without any work if the content type is not matched. It creates the possibility of chaining multiple body parsers. [\#102 ](https://github.com/marblejs/marble/pull/102)
* **@marblejs/middleware-body** - Fixed wrongly parsed `application/x-www-form-urlencoded` body types [\#101 ](https://github.com/marblejs/marble/pull/101)

## 2.0.0-rc.0 - 2019-02-03

`pre-release`

**Whats new?**

* **@marblejs/core** - improved type-inference of combined middlewares [\#88 ](https://github.com/marblejs/marble/pull/88)
* **@marblejs/core** - Effect response output stream [\#96](https://github.com/marblejs/marble/pull/96)
* **@marblejs/core** - `createServer` bootstrapping function [\#89](https://github.com/marblejs/marble/pull/89) [\#91 ](https://github.com/marblejs/marble/pull/91)
* **@marblejs/core** - improved type declaration of `combineRoutes` function
* **@marblejs/websockets** - WebSockets module [\#89 ](https://github.com/marblejs/marble/pull/89)
* **@marblejs/middlware-joi** - improved middleware type inference [\#88](https://github.com/marblejs/marble/pull/88)
* **@marblejs/middlware-io** - introducing new effect validation middleware based on **io-ts** library
* **@marblejs/middlware-body** - entrypoint allows to pass configuration object
* TypeScript v3.3.x support

**Breaking changes**

* **@marblejs/core** - Improved request type inference. `req.params`, `req.body`, `req.query` are by default of `unknown` type instead of `any` [\#88](https://github.com/marblejs/marble/pull/88)
* **@marblejs/core** - `httpListener` config properties renaming [\#92 ](https://github.com/marblejs/marble/pull/98)
* **@marblejs/core** -  The third argument is a common `EffectMetadata` object which can contain eventual error object or contextual injector. [\#94](https://github.com/marblejs/marble/pull/94)  [\#95](https://github.com/marblejs/marble/pull/95) 
* **@marblejs/core** [\#97 ](https://github.com/marblejs/marble/pull/97)
  * `Effect` -&gt; `HttpEffect`
  * `Middleware` -&gt; `HttpMiddlewareEffect`
  * `ErrorEffect` -&gt; `HttpErrorEffect`

**Deprecation warnings**

* Deprecated **@marblejs/middleware-joi** package. Use `@marblejs/middleware-io` instead. [\#99](https://github.com/marblejs/marble/pull/99) 
* Deprecated **@marblejs/middleware-logger** `loggerWithOpts$` entrypoint. [\#98 ](https://github.com/marblejs/marble/pull/98)

## 1.2.1 - 2018-11-05

**Whats new?**

* **@marblejs/core** - fixed an issue with missing `chalk` dependency \(issue: [\#77](https://github.com/marblejs/marble/issues/77)\)

## 1.2.0 - 2018-11-05

**Whats new?**

* **@marblejs/middleware-logger** - extended available API \(see: [middleware-logger](../api-reference/middleware-logger.md) chapter\)
* **@marblejs/core** - Corrected nullable values detection inside **Maybe** monad

{% hint style="warning" %}
From version **v1.2** the `logger$` entry point is marked as deprecated. Use `loggerWithOpts$` instead. From the version **v2.x** the old entry point will be swapped with newer implementation
{% endhint %}

## 1.1.1 - 2018-10-21

**Whats new?**

* **@marblejs/core** resolves [\#67](https://github.com/marblejs/marble/issues/67) \(path matching for trailing slash combined with wildcard parameter\)

## 1.1.0 - 2018-09-29

10 days ago we've got an official v1.0.0 release - this time we have another cool news to share with you!

**Whats new?**

* Introducing new package `@marblejs/middleware-jwt` - a HTTP requests authentication middleware based on [JWT](https://jwt.io) mechanism. 🚀 🎉 For more API reference please visit our official [documentation](https://marblejs.gitbook.io/marble/available-middlewares/authorizeusd).
* Resolved an issue in the scenario when the grouped middleware throws an error \(eg. when request is not authorized\) and connected _Effect_ endpoint unexpectedly catches the thrown error. See more details please visit \#73 
* Support for TypeScript `v3.1.x`
* _\[internal\]_ - Moved back from `webpack` to well-tested `tsc` compiler which makes our build artifacts more predictable and less error prone

## 1.0.0 - 2018-09-18

![We made it! &#x1F680; ](../.gitbook/assets/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f336f366f7a6f6d6a7763514a70647a3570362f67.gif)

## 1.0.0-rc.3 - 2018-09-17

**Whats new?**

* Integration with [path-to-regexp](https://github.com/pillarjs/path-to-regexp) library
* Support for wildcard parameter, eg. `/foo/:param*`. For more details see: [example implementation](https://github.com/marblejs/marble/blob/master/packages/%40integration/src/controllers/static.controller.ts)

```typescript
const getFile$ = EffectFactory
  .matchPath('/:dir*')
  .matchType('GET')
  .use(req$ => req$
    .pipe(
      map(req => req.params.dir as string),
      switchMap(readFile(STATIC_PATH)),
      map(body => ({ body }))
    ));
```

* Fixed a problem with incorrectly generated declaration files for `@marblejs/middleware-joi` package.

## 1.0.0-rc.2 - 2018-09-10

`pre-release`

**Whats new?**

* Fixed problem with incorrectly concatenated wildcard endpoint when combined with parametrized route.

```typescript
const notFound$ = EffectFactory
  .matchPath('*')
  .matchType('*')
  .use(req$ => req$.pipe(
    switchMap(() =>
      throwError(new HttpError('Route not found', HttpStatus.NOT_FOUND))
    )
  ));

export const api$ = combineRoutes(
  '/api/:version',
  [ notFound$ ],
);
```

* For non-TypeScript developers there was no validation made during app startup, eg. `EffectFactory` methods were not validated if developer provided wrong HTTP method to the `matchType`. Going to the expectations we introduced dedicated `CoreError` type used for throwing an package related error messages, eg. for notifying non-TypeScript developers if they made a mistake in the method arguments.

![](https://user-images.githubusercontent.com/7793430/45293153-a43a3500-b4f7-11e8-97c9-e7ead0846255.png)

* TypeScript v.3.0.x support
* RxJS v6.2.2 support
* Lerna v3.3.0 support
* Introduced Webpack based build process - from now builds are optimized and properly compressed

## 1.0.0-rc.1 - 2018-08-14

`pre-release`

**Breaking changes:**

* Changed `httpListener` error handler attribute from `errorMiddleware` 👉 `errorEffect`. There was an inconsistency with previous error handler definition and naming. We had to correct this because error handler acts as an _Effect_ instead of _Middleware_.

_Old bootstrapping API:_

```typescript
const app = httpListener({
  middlewares,
  effects,
  errorMiddleware,  👈
});
```

_New bootstrapping API:_

```typescript
const app = httpListener({
  middlewares,
  effects,
  errorEffect,  👈
});
```

**Whats new?**

* Exposed `res.send` method - from now you don't have to send response manually via dedicated Node.js `http.OutgoingMessage` API. The `res.send` method returns an empty stream, thus it can be easily composed inside middleware pipeline.

```typescript
const middleware$: Middleware = (req$, res) =>
  req$.pipe(
    switchMap(() => res.send({ body: 💩, status: 304, headers: /* ... */ }),
  );
```

* Exposed type aliases for common Marble.js architectural blocks:

```typescript
const effect$: Effect = req$ =>
  req$.pipe(
    // ...
  );

const middleware$: Middleware = (req$, res) =>
  req$.pipe(
    // ...
  );

const error$: ErrorEffect = (req$, res, err) =>
  req$.pipe(
    // ...
  );
```

## 1.0.0-rc.0 - 2018-07-07

`pre-release`

**Breaking changes**

* In order to factorize the routing table statically, we need to introduce the breaking change in **Effect** API definition. 

_Old Effect API:_

```typescript
const getUsers$: Effect = request$ =>
  request$.pipe(
    matchPath('/'),
    matchType('GET'),
    // ...
  );
```

_New Effect API:_

```typescript
const getUsers$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // ...
  ));
```

* separately imported `matchPath` and `matchType` stream operators are removed and are part of `EffectFactory` instead.

**Whats new?**

* **@marblejs/core** - **internals** - refactored + rebuilt routing resolving mechanism. Thanks to the latest changes we gained **hudge** performance boost.
* **@marblejs/core** - **internals** - improved performance for middleware resolving flow
* **@marblejs/core** - **internals** - rewritten URL params intercepting mechanism
* **@marblejs/middleware-body** - added support for _x-www-form-urlencoded_ Content-Type

## 0.5.0 - 2018-06-13

`pre-release`

**Fixes**

* corrected `@marblejs/middleware-logger` response time logging \(issue: [\#48](https://github.com/marblejs/marble/pull/48)\)

**Whats new?**

* `combineRoutes()` API allows to compose middlewares for grouped routes \(feature request: [\#36](https://github.com/marblejs/marble/issues/36)\) \(see [docs](../overview/routing.md#route-composition)\)

## 0.4.2 - 2018-06-10

`pre-release`

**Fixes**

* fixed an issue with not matched routes in case of reordered `matchType` and `matchPath`operators for the same routes but with different methods \(PR [\#46](https://github.com/marblejs/marble/pull/46)\)

## 0.4.1 - 2018-06-03

`pre-release`

**Fixes**

* resolved an issue with TypeScript compiler flag responsible for strict function types \(issue: [\#43](https://github.com/marblejs/marble/issues/43)\)
* added support for TypeScript `v2.9.1`

## 0.4.0 - 2018-05-31

`pre-release`

**Fixes**

* fixed an issue with _Effects_ matching hazard \(issue: [\#35](https://github.com/marblejs/marble/issues/35)\)

**Whats new?**

* ability to create 404 handlers using `matchType('*')` and `matchPath('*')` \(issue: [\#27](https://github.com/marblejs/marble/issues/27)\)
* `HttpError` constructor now can take `data: object` as optional third argument

## 0.3.2 - 2018-05-29

`pre-release`

**Fixes**

* support for non-JSON _Effect_ return types \(issue:[ \#33](https://github.com/marblejs/marble/issues/33)\) **Thanks** [**@couzic**](https://github.com/couzic)**!**

**Whats new?**

* `mime-type` / `content-type` auto detection

## 0.3.1 - 2018-05-27

`pre-release`

**Fixes**

* corrected  _peerDependency versions_ for `@marblejs/*` packages

## 0.3.0 - 2018-05-27

`pre-release`

**Breaking changes**

* moved `bodyParser$` and `logger$` middleware outside `@marblejs/core` to separete`@marblejs/middleware-body` and `@marblejs/middleware-logger` packages

**Whats new?**

* added support for intercepting path parameters `req.params` \(see: [Routing](../overview/routing.md#route-parameters) chapter\) 
* added support for intercepting query parameters `req.query` \(see: [Routing](../overview/routing.md#route-query-parameters) chapter\)
* ability to compose middlewares inside Effect request pipeline via use operator \(see: [Middlewares](../overview/middlewares.md#middlewares-composition) chapter\)
* new middleware `@marblejs/middleware-joi` - a Joi [validation middleware](../api-reference/middleware-joi.md) 
* minor bugfixes and code improvements

## 0.2.x - 2018-05-14

`pre-release`

Initial pre-release version



