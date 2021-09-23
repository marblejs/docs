---
description: Effect operator for composing middleware directly inside stream pipeline.
---

# operator: use

{% hint style="info" %}
Since version 4.0 `use` operator is deprecated. You can easily compose middlewares directly to the Observable chain.
{% endhint %}

### **Importing**

```typescript
import { use } from '@marblejs/core';
```

### **Type declaration**

```typescript
use :: <I, O>(MiddlewareLike<I, O>, <?>EffectContext) -> Observable<I>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _middleware_ | `MiddlewareLike` |
| ctx  | _&lt;optional&gt;_ `EffectContext` |

### **Returns**

`Observable<I>`

### Example

```typescript
import { use } from '@marblejs/core';
import { r } from '@marblejs/http';

const foo$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffet(req$ => req$.pipe(
    // ...
    use(authorize$),
    // ...
  )));
```

