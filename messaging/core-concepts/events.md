# Events

The messaging module builds the mental model around an event - a uniform structure that represents a notification of specific type. Events can be handled in a variety of ways. For example, they can be published to a queue that guarantees delivery of the event to the appropriate consumers, or they can be published to a “pub/sub” model stream that publishes the event and allows access to all interested parties. In either case, the _producer_ sends the event, and the _consumer_ receives that event, reacting accordingly.

### Interface

Events are plain JavaScript objects. They must have a `type` property that indicates the type of event being performed and optional `payload` that carries additional information needed to process the event. As in most messaging systems, events are reified, i.e., they are represented as concrete objects and they can be stored and passed around.

Messaging mental model is a very simple, so it does not make a lot of assumptions about your events. Marble.js does not prescribe one way to construct them, nor it tells us how to define types and payloads. But this does not mean that all events are alike  👉 [CQRS](../cqrs.md).

```typescript
import { Event } from '@marblejs/core';

const event: Event = {
  type: 'USER_CREATED',
  payload: { id: '#123' },
};
```

Marble.js events can also carry additional information in form of  `metdata` that can be treat as less important from the domain perspective. They can include their unique/correlation identifier which can be used for determining to which input event the output event belongs. Besides that, they can also carry the information about a channel to which the response event should be directed. Both mentioned properties can be used for RPC \(Remote Procedure Call\) messaging.

Additionally, the interface defines an `error` property which can be used to optionally carry the exception information in form of plain object. The last and less important property is used internally by the framework to carry the `raw` data about the incoming event in form of transport layer compatible object.

```typescript
interface Event<P, E, T> {
  type: T;
  payload?: P;
  error?: E;
  metadata?: EventMetadata;
}

```

```typescript
interface EventMetadata {
  correlationId?: string;
  replyTo?: string;
  raw?: any;
}
```

{% hint style="info" %}
The design decision behind Marble.js messaging assumes that all events are serialized to the plain form that can be easily transferred via the underlying I/O transport layer. This means that _Date_ objects or other specialized instances will be serialized to the simpler form, eg. to string or object records.
{% endhint %}

### I/O event decoding/encoding

Event-based communication follows the same laws as request-based communication - each incoming event should be validated before usage. With an introduction of version 3.3.0, **@marblejs/core** module exposes a dedicated **event** decoder/encoder which helps with maintaining a set of event codecs thanks to everyone well known `io-ts` library.

{% tabs %}
{% tab title="user.events.ts" %}
```typescript
import { event } from '@marblejs/core';
import * as t from 'io-ts';

enum UserEventType {
  USER_CREATED = 'USER_CREATED',
  USER_UPDATED = 'USER_UPDATED',
};

export const UserCreatedEvent =
  event(UserEventType.USER_CREATED)(t.type({
    id: t.string,
  }));
  

export const UserUpdatedEvent =
  event(UserEventType.USER_UPDATED)(t.type({
    id: t.string,
  }));
```
{% endtab %}
{% endtabs %}

Thanks to `io-ts` library we can infer event type from its definition and construct the event object accordingly:

```typescript
/*
const userUpdatedEvent: {
  type: UserEventType.USER_UPDATED,
  payload: {
    id: string,
  },
}
*/
const userUpdatedEvent = UserUpdatedEvent.create({ 
  id: 'some_id',
});
```

In order to match the incoming event agains the type literal and validator all you have to do is to apply the event codec to `matchEvent` operator and `eventValidator$` middleware.

```typescript
import { act, matchEvent } from '@marblejs/core';
import { MsgEffect } from '@marblejs/messaging';
import { eventValidator$ } from '@marblejs/middleware-io';
import { UserUpdatedEvent } from './user.events';

const userUpdated$: MsgEffect = event$ =>
  event$.pipe(
    matchEvent(UserUpdatedEvent),
    act(eventValidator$(UserUpdatedEvent)),
    act(event => ...),
  );
```

