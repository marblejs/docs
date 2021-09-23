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
import { matchEvent } from '@marblejs/core';
import { HttpServerEffect, ServerEvent } from '@marblejs/http';
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
import { matchEvent, event, EventsUnion } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import * as t from 'io-ts';
import { map } from 'rxjs/operators';

// Commands definition

export enum OfferCommandType {
  GENERATE_OFFER_DOCUMENT = 'GENERATE_OFFER_DOCUMENT',
}

export const GenerateOfferDocumentCommand =
  event(OfferCommandType.GENERATE_OFFER_DOCUMENT)(t.type({ offerId: t.string }));

// Effect definition

const generateOfferDocument$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent(GenerateOfferDocumentCommand),
    map(event => event.payload), // (typeof payload) = { offerId: string };
    // ...
  );
```

