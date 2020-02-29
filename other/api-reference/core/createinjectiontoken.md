---
description: >-
  A lookup token associated with a dependency provider, for use with the
  Marble.js context system.
---

# createContextToken

### **Importing** <a id="importing"></a>

```typescript
import { createContextToken } from '@marblejs/core';
```

### **Type declaration**

```text
createContextToken :: () -> ContextToken
createContextToken :: string -> ContextToken
```

### **Example**

The following example creates and associates injection token with Marble.js WebSocket server.

```typescript
import { createContextToken } from '@marblejs/core';
import { WebSocketServerConnection } from '@marblejs/websockets';

const WebSocketServerToken =
    createContextToken<WebSocketServerConnection>('WebSocketServerToken');
```

{% hint style="info" %}
Always remember add a string name for created lookup token. 
{% endhint %}



