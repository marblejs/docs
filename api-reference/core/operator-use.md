---
description: Effect operator for composing middleware directly inside stream pipeline.
---

# operator: use

### **Importing**

```typescript
import { use } from '@marblejs/core';
```

### **Type declaration**

```typescript
use :: <I, O>(MiddlewareLike<I, O>, <?>any, <?>any) -> Observable<I>
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _middleware_ | `MiddlewareLike` |
| _client_  | &lt;_optional_&gt; protocol specific client instance |
| _meta_ | _&lt;optional&gt;_ `EffectMetadata` |

### **Returns**

`Observable<I>`

### Example

```typescript
import { r, use } from '@marblejs/core';

const foo$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffet(req$ => req$.pipe(
    // ...
    use(authorize$),
    // ...
  )));
```

