# Changelog

## 1.2.1 - 2018-11-05

**Whats new?**

* `@marblejs/core` - fixed an issue with missing `chalk` dependency \(issue: [\#77](https://github.com/marblejs/marble/issues/77)\)

## 1.2.0 - 2018-11-05

**Whats new?**

* `@marblejs/middleware-logger` - extended available API \(see: [middleware-logger](available-middlewares/logger.md) chapter\)
* `@marblejs/core` - Corrected nullable values detection inside **Maybe** monad

{% hint style="warning" %}
From version **v1.2** the `logger$` entry point is marked as deprecated. Use `loggerWithOpts$` instead. From the version **v2.x** the old entry point will be swapped with newer implementation
{% endhint %}

## 1.1.1 - 2018-10-21

**Whats new?**

* `@marblejs/core` resolves [\#67](https://github.com/marblejs/marble/issues/67) \(path matching for trailing slash combined with wildcard parameter\)

## 1.1.0 - 2018-09-29

10 days ago we've got an official v1.0.0 release - this time we have another cool news to share with you!

**Whats new?**

* Introducing new package `@marblejs/middleware-jwt` - a HTTP requests authentication middleware based on [JWT](https://jwt.io) mechanism. 🚀 🎉 For more API reference please visit our official [documentation](https://marblejs.gitbook.io/marble/available-middlewares/authorizeusd).
* Resolved an issue in the scenario when the grouped middleware throws an error \(eg. when request is not authorized\) and connected _Effect_ endpoint unexpectedly catches the thrown error. See more details please visit \#73 
* Support for TypeScript `v3.1.x`
* _\[internal\]_ - Moved back from `webpack` to well-tested `tsc` compiler which makes our build artifacts more predictable and less error prone

## 1.0.0 - 2018-09-18

![We made it! &#x1F680; ](.gitbook/assets/68747470733a2f2f6d656469612e67697068792e636f6d2f6d656469612f336f366f7a6f6d6a7763514a70647a3570362f67697068792e676966.gif)

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

* `@marblejs/core` - **internals** - refactored + rebuilt routing resolving mechanism. Thanks to the latest changes we gained **hudge** performance boost.
* `@marblejs/core` - **internals** - improved performance for middleware resolving flow
* `@marblejs/core` - **internals** - rewritten URL params intercepting mechanism
* `@marblejs/middleware-body` - added support for _x-www-form-urlencoded_ Content-Type

## 0.5.0 - 2018-06-13

`pre-release`

**Fixes**

* corrected `@marblejs/middleware-logger` response time logging \(issue: [\#48](https://github.com/marblejs/marble/pull/48)\)

**Whats new?**

* `combineRoutes()` API allows to compose middlewares for grouped routes \(feature request: [\#36](https://github.com/marblejs/marble/issues/36)\) \(see [docs](basics/routing.md#route-composition)\)

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

_`pre-release`_

**Fixes**

* corrected  _peerDependency versions_ for `@marblejs/*` packages

## 0.3.0 - 2018-05-27

_`pre-release`_

**Breaking changes**

* moved `bodyParser$` and `logger$` middleware outside `@marblejs/core` to separete`@marblejs/middleware-body` and `@marblejs/middleware-logger` packages

**Whats new?**

* added support for intercepting path parameters `req.params` \(see: [Routing](basics/routing.md#route-parameters) chapter\) 
* added support for intercepting query parameters `req.query` \(see: [Routing](basics/routing.md#route-query-parameters) chapter\)
* ability to compose middlewares inside Effect request pipeline via use operator \(see: [Middlewares](basics/middlewares.md#middlewares-composition) chapter\)
* new middleware `@marblejs/middleware-joi` - a Joi [validation middleware](available-middlewares/joi.md) 
* minor bugfixes and code improvements

## 0.2.x - 2018-05-14

_`pre-release`_

Initial pre-release version



