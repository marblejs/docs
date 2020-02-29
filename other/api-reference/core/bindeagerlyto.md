---
description: Binds injection token to eager dependency.
---

# bindEagerlyTo

### **Importing** <a id="importing"></a>

```typescript
import { bindEagerlyTo } from '@marblejs/core';
```

### **Type declaration**

```text
bindEagerlyTo ::  ContextToken -> ContextReader -> BoundDependency

```

The function is responsible for binding context token to `ContextReader` which can be a **fp-ts** `Reader` instance or plain **sync/async** `() => T`. The function is producing a eager binding, which means that, the dependency will be evaluated on startup.

### **Example**

```typescript
import { reader, bindEagerlyTo, createServer, createContextToken } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import * as R from 'fp-ts/lib/Reader';

// create sync reader

const FooSyncToken = createContextToken<ReturnType<typeof fooSync>>('FooSyncToken');

const fooSync = pipe(reader, R.map(ask => {
  const otherDependency = ask(OtherToken);
  
  return { ... };
});

// create async reader

const FooAsyncToken = createContextToken<ReturnType<typeof fooSync>>('FooSyncToken');

const fooAsync = pipe(reader, R.map(async ask => {
  const otherDependency = ask(OtherToken);
  
  return { ... };
});


// bind readers to tokens

const boundSyncDependency  = bindEagerlyTo(FooSyncToken)(fooSync);
const boundAsyncDependency = bindEagerlyTo(FooAsyncToken)(fooAsync);

```



