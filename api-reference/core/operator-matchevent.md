---
description: Effect operator for matching incoming events.
---

# operator: matchEvent

### Importing

```typescript
import { matchEvent } from '@marblejs/core';
```

### **Type declaration**

```typescript
matchEvent :: (EventLike | EventCreator) -> Observable<Event> -> Observable<Event>
```

### Example

_**WebSockets:**_

```typescript
import { matchEvent } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';

const add$: WsEffect = event$ =>
  event$.pipe(
    matchEvent('ADD'),
    map(event => event.payload), // (typeof payload) = unknown
    // ...
  );
```

_**HttpServerEffect:**_

```typescript
import { matchEvent, HttpServerEffect, ServerEvent } from '@marblejs/core';

const listening$: HttpServerEffect = event$ =>
  event$.pipe(
    matchEvent(ServerEvent.listening),
    map(event => event.payload), // (typeof payload) = { port: number; host: string; }
    // ...
  );
```

