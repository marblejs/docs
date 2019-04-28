# Effects

## **Hello, world!**

**Effect** is the main building block of the whole framework. It is just a function which returns a stream of events. Using its generic interface we can define API endpoints, [middlewares](middlewares.md), [error handlers](error-handling.md) and much more \(see next chapters\). Let's define our first "hello world" `HttpEffect`!

```typescript
const helloEffect$: HttpEffect = req$ => req$.pipe(
  mapTo({ body: 'Hello, world!' }),
);
```

The _Effect_ above responds to incoming request with `Hello, world!` message. In Marble.js, every _Effect_ tries to be referentailly transparent, which means that each incoming request has to be mapped to object with attributes like `body`, `status` or `headers`. If the _status_ code or _headers_ are not defined, then API by default responds with _200_ status and _application/json_ header.

In order to route our first _Effect,_ we have to define the path and HTTP method that the incoming request should be matched to. The simplest implementation of HTTP API endpoint can look like in example below.

{% code-tabs %}
{% code-tabs-item title="effect-with-pipe.ts" %}
```typescript
const hello$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(helloEffect$));
```
{% endcode-tabs-item %}

{% code-tabs-item title="effect-with-EffectFactory.ts" %}
```typescript
const hello$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    mapTo({ body: `Hello, world!` })
  ));
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Lets define a little bit more complex endpoint.

{% code-tabs %}
{% code-tabs-item title="postUser.effect.1.ts" %}
```typescript
const postUser$ = r.pipe(
  r.matchPath('/user'),
  r.matchType('POST'),
  r.useEffect(req$ => req$.pipe(
    map(req => req.body as User),
    mergeMap(Dao.postUser),
    map(response => ({ body: response }))
  )));
```
{% endcode-tabs-item %}

{% code-tabs-item title="postUser.effect.2.ts" %}
```typescript
const postUser$ = EffectFactory
  .matchPath('/user')
  .matchType('POST')
  .use(req$ => req$.pipe(
    map(req => req.body),
    mergeMap(Dao.postUser),
    map(response => ({ body: response }))
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

The example above will match every _POST_ request that matches to `/user` url. Using previously parsed _POST_ body \(see [bodyParser$](../api-reference/middleware-body.md) middleware\) we can map it to example _DAO_ which returns a `HttpEffectResponse` object as an action confirmation.

{% hint style="info" %}
Since Marble.js 2.0, you can build HTTP API routes using [`EffectFactory`](../api-reference/core/core-effectfactory.md) builder or using new, pipeable functions inside[`r.pipe`](../api-reference/core/r.pipe.md) function.
{% endhint %}

## HttpRequest

Every _HttpEffect_ has an access to two most basics objects created by _http.Server_. **HttpRequest** is an abstraction over basic _Node.js_ [http.IncomingMessage](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage) object. It may be used to access response status, headers and data, but in most scenarios you don't have to deal with all available APIs offered by _IncomingMessage_ class. The most common properties available in request object are:

* _url_
* _method_
* _headers_
* _body \(see_ [_bodyParser$_](../api-reference/middleware-body.md) _section\)_
* _params \(see_ [_routing_](routing.md) _chapter\)_
* _query \(see_ [_routing_](routing.md) _chapter\)_

{% hint style="info" %}
For more details about available API offered in `http.IncomingMessage`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage).
{% endhint %}

## HttpResponse

Like previously described _HttpRequest_, the **HttpResponse** object is also an abstraction over basic _Node.js_ [http.ServerResponse](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse) object. Besides the default API, the response object exposes an `res.send` method, which can be a handy wrapper over _Marble.js_ responding mechanism. For more detailed information about _res.send_ method, visit [Middlewares](middlewares.md#sending-a-response-earlier) chapter.

{% hint style="info" %}
For more details about available API offered in `http.ServerResponse`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse).
{% endhint %}

