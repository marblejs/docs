# Error handling

## Error handling

The event nature of WebSocket protocol defines how app errors should be handled. Once a connection has been established between the client and the server, communitation between these two points should be handled only through events - that also concerns errors.

{% hint style="danger" %}
Avoid throwing exceptions explicitly when possible. If the exception is thrown, try to catch it and send _Event_ instead.
{% endhint %}

```typescript
export const saveUser$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('SAVE_USER'),
    map(event => event.payload),
    mergeMap(Dao.saveUser),
    catchError(error => of({
      type: 'SAVE_USER_ERROR',
      payload: error.message,
    })),
  );
```

If you are building a reusable middleware or you really need to throw an exception, [@marblejs/core](../api-reference/core/) exports a dedicated event error constructor which can be intercepted via listener error handler. The `EventError` constructor takes as a first parameter an event object and as a second parameter a string message. Optionally you can pass the third argument, which can contain any other error related value.

```typescript
new EventError(event, 'some error message', additionalData);
```

## Custom error handling effect

By default _Marble.js_ comes with simple and lightweight error handling effect, but you can define a custom one if you want.

{% code-tabs %}
{% code-tabs-item title="error.ws-effect.ts" %}
```typescript
export const customError$: WsErrorEffect<ThrownError> = (event$, client, meta) =>
  req$.pipe(
    map(event => ({
      type: 'ERROR',
      payload: meta.error.message
    }),
  );
```
{% endcode-tabs-item %}
{% endcode-tabs %}

As any other Effect, error handler maps the stream of errored events to objects of type _Event_ \(`type`, `payload`\). The _WsErrorEffect_ can retrieve from the third argument an intercepted error object which can be used for error handling-related logic.

To connect the custom error effect, all you have to do is to attach it to `error$` property in `webSocketListener` config object.

{% code-tabs %}
{% code-tabs-item title="webSocket.listener.ts" %}
```typescript
import { webSocketListener } from '@marblejs/websockets';
import { customError$ } from './error.ws-effect';

const webSocketServer = webSocketListener({
  middlewares,
  effects,
  // Custom error effect 👇
  error$: customError$,
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

