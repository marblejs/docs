---
description: >-
  Every Marble.js listener factory allows you to intercept outgoing messages via
  dedicated output$ handler.
---

# Output

## Building your own output effect

Using `HttpOutputEffect` you can grab an outgoing HTTP response and map it into different response, modifying outgoing data on demand.

```haskell
HttpOutputEffect :: Observable<{ req: HttpRequest; res: HttpEffectResponse }>
  -> Observable<HttpEffectResponse>
```

`HttpOutputEffect` allows you to grab the outgoing response together with corresponding initial request. Lets build a simple response compression middleware.

{% tabs %}
{% tab title="output.effect.ts" %}
```typescript
import { HttpOutputEffect } from '@marblejs/core';
import { map } from 'rxjs/operators';
import * as zlib from 'zlib';

const output$: HttpOutputEffect = res$ =>
  res$.pipe(
    map(({ res, req }) =>  {
      switch(req.headers['accept-encoding']) {
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
{% endtab %}
{% endtabs %}

To connect the output effect, all you need to do is to attach it to `output$` property in `httpListener` config object.

```typescript
import { httpListener } from '@marblejs/core';
import { output$ } from './output.effect';

export const listener = httpListener({
  middlewares: [ ... ],
  effects: [ ... ],
  output$, // 👈
});
```

