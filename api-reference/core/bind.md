---
description: Binds injection token to dependency.
---

# bindTo

### **Importing** <a id="importing"></a>

```typescript
import { bindTo } from '@marblejs/core';
```

### **Type declaration**

```text
bindTo ::  ContextToken -> ContextDependency -> BoundDependency
```

### **Example**

Bind context token to basic types \(eg. object\):

```typescript
import { reader, bindTo, createServer, createContextToken } from '@marblejs/core';

const config: Config = { /* ... */ };
const configReader = reader.map(() => config);
const Token = createContextToken<Config>();

// ----------------

createServer({
  // ...
  dependencies: [
    bindTo(Token)(configReader),
  ],
});
```

Bind context token to injectable factory function:

```typescript
import { reader, bindTo, createServer, createContextToken } from '@marblejs/core';

const fooReader = reader.map(ctx => {
  const dependency = context.get(...);
  // ...
};
const Token = createContextToken<Foo>();

// ----------------

createServer({
  // ...
  dependencies: [
    bindTo(Token)(fooReader);
  ],
});
```

