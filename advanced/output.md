# Output interceptor

[httpListener](../api-reference/core/core-httplistener.md) and [webSocketListener](../api-reference/websockets/websocketlistener.md) allows to intercept outgoing messages via `output$` attribute. Using `HttpOutputEffect` or `WsOutputEffect` you can grab an outgoing HTTP/Event response and map it into another form of outgoing message. Using this kind of response interceptor you can modify certain type of responses on demand. For example, in case of `Accept: application/vnd.api+json`  you would like to adjust the response accordingly to fit the needs of JSON API specification.

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

HTTP output interceptor allows you to grab the initial request via `EffectMetadata.initiator`. You can use it eg. for creating a compression middleware.

```typescript
import { HttpOutputEffect } from '@marblejs/core';

const output$: HttpOutputEffect = (res$, _, { initiator }) =>
  res$.pipe(
    map(res =>  {
      switch(initiator.headers['accept-encoding']) {
        case 'br':
          return ({
            ...res,
            headers: { ...res.headers, 'Content-Encoding': 'br' },
            body: res.body.pipe(zlib.createBrotliDecompress()),
          });
        case 'gzip':
          return ({
            ...res,
            headers: { ...res.headers, 'Content-Encoding': 'gzip' },
            body: res.body.pipe(zlib.createGunzip()),
          });
        case 'deflate':
          return ({
            ...res,
            headers: { ...res.headers, 'Content-Encoding': 'deflate' },
            body: res.body.pipe(zlib.createInflate()),
          });
        default:
          return res;
      }
    }),
  );
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

