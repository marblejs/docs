---
description: Effect operator for composing middleware directly inside request pipeline.
---

# operator: use

### **Importing**

```typescript
import { use } from '@marblejs/core';
```

### **Type declaration**

```typescript
use :: (Middleware, <?>HttpResponse) -> Observable<HttpRequest>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _middleware_ | `Middleware` \(`Effect<HttpRequest>`\) effect |
| _response_  | &lt;_optional_&gt; `HttpResponse` object |

### **Returns**

`Observable<HttpRequest>`

### Example

```typescript
import { use } from '@marblejs/core';

const foo$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    // ...
    use(authorize$),
    // ...
  ));
```

