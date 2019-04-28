# Output interceptor

[httpListener](../api-reference/core/core-httplistener.md) and [webSocketListener](../api-reference/websockets/websocketlistener.md) allows to intercept outgoing messages via `output$` attribute. Using `HttpOutputEffect` or `WsOutputEffect` you can grap an outgoing HTTP/Event response and map it into another form of outgoing message. Using this kind of response interceptor you can modify certain type of responses on demand. For example, in case of `Accept: application/vnd.api+json`  you would like to adjust the response accordingly to fit the needs of JSON API specification.

**HTTP:**

```typescript
import { HttpOutputEffect, httpListener } from '@marblejs/core';

const output$: HttpOutputEffect = res$ =>
  res$.pipe(
    map(res =>  ... ),
  );
  
export default httpListener({
  middlewares: [ ... ],
  effects: [ ... ],
  output$,
});
```

**WebSockets:**

```typescript
import { WsOutputEffect, webSocket } from '@marblejs/websockets';

const output$: WsOutputEffect = event$ =>
  event$.pipe(
    map(event =>  ... ),
  );
  
export default webSocketListener({
  middlewares: [ ... ],
  effects: [ ... ],
  output$,
});
```

