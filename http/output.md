---
description: >-
  Every Marble.js listener factory allows you to intercept outgoing messages via
  dedicated output$ handler.
---

# Output

## Building your own output effect

Using `HttpOutputEffect` you can grab an outgoing Effect response and map it into different one, modifying outgoing data on demand.

```haskell
type Out = {
  request: HttpRequest;
  headers: HttpHeaders;
  status: HttpStatus;
  body: any;
};

HttpOutputEffect :: Observable<Out> -> Observable<Out>
```

`HttpOutputEffect` allows you to grab the outgoing response together with corresponding initial request.  The output effect works similar to the HttpMiddlewareEffect, but in that case for outgoing responses. Let's build a simple response compression middleware.

{% tabs %}
{% tab title="output.effect.ts" %}
```typescript
import { HttpOutputEffect } from '@marblejs/http';
import { map } from 'rxjs/operators';
import * as zlib from 'zlib';

const output$: HttpOutputEffect = res$ =>
  res$.pipe(
    map(({ request, headers, body, status }) =>  {
      switch(request.headers['accept-encoding']) {
        case 'br':
          return ({
            request,
            status,
            headers: { ...headers, 'Content-Encoding': 'br' },
            body: body.pipe(zlib.createBrotliDecompress()),
          });
        case 'gzip':
          return ({
            request,
            status,
            headers: { ...headers, 'Content-Encoding': 'gzip' },
            body: body.pipe(zlib.createGunzip()),
          });
        case 'deflate':
          return ({
            request,
            status,
            headers: { ...headers, 'Content-Encoding': 'deflate' },
            body: body.pipe(zlib.createInflate()),
          });
        default:
          return { status, headers, body, request };
      }
    }),
  );
```
{% endtab %}
{% endtabs %}

To connect the output effect, all you need to do is to attach it to `output$` property in `httpListener` config object.

```typescript
import { httpListener } from '@marblejs/http';
import { output$ } from './output.effect';

export const listener = httpListener({
  middlewares: [ ... ],
  effects: [ ... ],
  output$, // 👈
});
```
