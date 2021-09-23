---
description: Binds injection token to lazy dependency.
---

# bindTo

### **Importing** <a id="importing"></a>

```typescript
import { bindTo, bingLazilyTo } from '@marblejs/core';
```

### **Type declaration**

```text
bindTo ::  ContextToken -> ContextReader -> BoundDependency
bindLazilyTo ::  bindTo
```

The function is responsible for binding context token to `ContextReader` which can be a **fp-ts** `Reader` instance or plain a `() => T`. The function is producing a lazy binding, which means that, the dependency will be evaluated on it's first call \(injection\).

`bindTo` is an alias of `bindLazilyTo`. 

### **Example**

```typescript
import { reader, bindTo, createContextToken } from '@marblejs/core';
import { createServer } from '@marblejs/http';
import { pipe } from 'fp-ts/lib/function';
import * as R from 'fp-ts/lib/Reader';

// create reader

const FooToken = createContextToken<Foo>('FooToken');

const foo = pipe(reader, R.map(ask => {
  const otherDependency = ask(OtherToken);
  
  return { ... };
});


// bind reader to token

const boundDependency = bindTo(FooToken)(foo);
```



