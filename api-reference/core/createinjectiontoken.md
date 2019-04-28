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
```

### **Example**

The following example creates and associates injection token with Marbke.js WebSocket server.

```typescript
import { createContextToken } from '@marblejs/core';
import { MarbleWebSocketServer } from '@marblejs/websockets';

export const WebSocketServerToken = createContextToken<MarbleWebSocketServer>();
```

