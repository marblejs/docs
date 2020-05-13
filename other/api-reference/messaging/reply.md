---
description: Constructs a reply-event
---

# reply

### **Importing** <a id="importing"></a>

```typescript
import { reply } from '@marblejs/messaging';
```

### **Type declaration**

```haskell
reply :: string -> Event -> Event
reply :: EventMetadata -> Event -> Event
reply :: Event -> Event -> Event
```

The function is responsible for constructing a reply-event for a channel name, EventMetadata object or origin event.

### **Example**

**Reply to channel:**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { reply, MsgEffect } from '@marblejs/messaging';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => reply('event_bus')({ type: 'FOO_RESPONSE' })),
  );
```

**Reply with metadata construct:**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { reply, MsgEffect } from '@marblejs/messaging';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => reply({
      replyTo: event.metadata.replyTo,
      correlationId: event.metadata.correlationId,
    })({ type: 'FOO_RESPONSE' })),
  );
```

**Reply to event:**

```typescript
import { act, matchEvent } from '@marblejs/core';
import { reply, MsgEffect } from '@marblejs/messaging';

const foo$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent('FOO'),
    act(event => reply(event)({ type: 'FOO_RESPONSE' })),
  );
```

