# FAQ

## Does Marble.js uses Express underneath?

No! Express is a cool library but it shows its age. From the very beginning the aim of Marble.js was to build it from scratch with its own philosophy in mind. Thanks to the design decisions that were set at the beginning of its existence, Marble.js is faster than Express and comparable to the most performant HTTP libraries available on the Node.js platform.

## Why should I choose Marble.js?

There are many alternatives available on Node.js platform. Marble.js defines a uniform interface for asynchronous processing of incoming events, which makes the processing easy to adapt to different forms of transport protocols. Marble.js fits best in event-based architectures, like: CQRS, Event Sourcing or microservices, but you can also build a CRUD-like systems with ease without touching previously mentioned concepts. Besides that, if you like of functional programming you should definitely check it out.

## Why the heck I need here RxJS?

Despite the single event nature of basic HTTP, there are no contradictions against using it for single events, like in HTTP. Marble.js uses RxJS as a hammer for expressing asynchronous flow with monadic manner, even if you have to deal with only one event. Marble.js doesn't operate only over basic [HTTP](../http/effects.md) protocol but can be used also for both [WebSocket](../messaging/websockets.md) and event-based [messaging](../messaging/core-concepts/effects.md) patterns, where the multi-event nature fits best. Don't be scared of the complexity and abstractions presented in RxJS API — the Marble.js framework, in general, is incredibly simple.

## Does Marble.js has a built-in ORM support?

As a design decision, the framework focus areas are oriented around request/message processing, which means that Marble.js tries to be database agnostic in any aspect, allowing you to easily integrate with any SQL or NoSQL database provider or ORM framework. You have a number of options available to you, depending on your preferences.

## How to use Marble.js with GraphQL?

There is no official module for GraphQL yet. Sorry.



