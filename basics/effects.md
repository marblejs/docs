# Effects

## **Hello, world!**

**Effect** is the main building block of the whole framework. Using its generic interface we can define API endpoints \(so called: _Effects_\), [middlewares](middlewares.md) and [error handlers](error-handling.md) \(see next chapters\).

The simplest implementation of API endpoint can look like this:

{% code title="" %}
```typescript
const helloWorld$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    mapTo({ body: `Hello, world!` })
  ));
```
{% endcode %}

The sample _Effect_ above resolves every _HTTP_ request that matches to root `/` path and _GET_ method type and at the end responds with `Hello, world!` message. Simple as hell, right?

Every API _Effect_ request has to be mapped to object which can contain attributes like `body`, `status` or `headers`. If _status_ code or _headers_ are not passed, then API by default will respond with _200_ status and `application/json` _Content -Type_ header.

A little bit more complex example can look like this:

{% code title="postUser.effect.ts" %}
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
{% endcode %}

The example above will match every _POST_ request that matches to `/user` url. Using previously parsed _POST_ body \(see [bodyParser$](../available-middlewares/body.md) middleware\) we can map it to sample _DAO_ which returns a `response` object as an action confirmation.

## HttpRequest

Every _Effect_ has an access to two most basics objects created by _http.Server_. The first one is **HttpRequest** which is just an abstraction over basic _Node.js_ [http.IncomingMessage](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage) object. It may be used to access response status, headers and data, but in most scenarios you don't have to deal with all available APIs offered by _IncomingMessage_ class. The most common properties available in request object are:

* _url_
* _method_
* _body \(see_ [_$bodyParser_](../available-middlewares/body.md) _section\)_
* _params \(see_ [_routing_](routing.md) _chapter\)_
* _query \(see_ [_routing_](routing.md) _chapter\)_

{% hint style="info" %}
For more details about available API offered in `http.IncomingMessage`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage).
{% endhint %}

## HttpResponse

Like previously described _HttpRequest_, the **HttpResponse** object is also just an abstraction over basic _Node.js_ [http.ServerResponse](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse) object. Besides the default API, the response object exposes an `res.send` method, which can be a handy wrapper over _Marble.js_ responding mechanism. For more detailed information about _res.send_ method, visit [Middlewares](middlewares.md#sending-a-response-earlier) chapter.

{% hint style="info" %}
For more details about available API offered in `http.ServerResponse`, please visit [official Node.js docummentation](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse).
{% endhint %}

