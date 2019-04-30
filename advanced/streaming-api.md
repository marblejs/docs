# Streaming

Streams are collections of data that might not be available all at once, and they don’t have to fit in memory. This makes streams really powerful when working with large amounts of data, or data that’s coming from an external source one _chunk_ at a time.

Since Marble.js **v2.1** you can send directly Node.js streams through _HttpEffectResponse_ `body` attribute which will be piped internally as client response.

```typescript
import { r, combineRoutes, use } from '@marblejs/core';
import { requestValidator$, t } from '@marblejs/middleware-io';
import { readFile } from '@marblejs/core/dist/+internal';

const getFileValidator$ = requestValidator$({
  params: t.type({ dir: t.string })
});

const getFile$ = r.pipe(
  r.matchPath('/static/:dir*'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    use(getFileValidator$),
    map(req => req.params.dir),
    map(dir => fs.createReadStream(path.resolve(STATIC_PATH, dir))),
    map(body => ({ body }))
  )),
);
```

