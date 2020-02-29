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

_**WsEffect:**_

```typescript
import { matchEvent } from '@marblejs/core';
import { WsEffect } from '@marblejs/websockets';
import { map } from 'rxjs/operators';

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
import { map } from 'rxjs/operators';

const listening$: HttpServerEffect = event$ =>
  event$.pipe(
    matchEvent(ServerEvent.listening),
    map(event => event.payload), // (typeof payload) = { port: number; host: string; }
    // ...
  );
```

_**MsgEffect:**_

```typescript
import { matchEvent, createEvent, EventsUnion } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { map } from 'rxjs/operators';

// Commands definition

export enum AddCommandType {
  ADD = 'ADD',
};

export const AddCommand = {
  add: createEvent(
    AddCommandType.ADD,
    (val: number) => ({ val }),
  ),
};

export type AddCommand = EventsUnion<typeof AddCommand>;

// Effect definition

const add$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent(AddCommand.add),
    map(event => event.payload), // (typeof payload) = { val: number };
    // ...
  );
```

