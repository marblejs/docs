---
description: >-
  The core idea of @marblejs/messaging module is focused around reacting to
  incoming events.
---

# Core concepts

## Overview

While you might have used REST as your service communications layer in the past, more and more projects are moving to an event-driven architecture. When a service performs some piece of work that other services might be interested in, that service produces an _event_ — a record of the performed action. Other services consume those events so that they can perform any of their own tasks needed as a result of the event.

Since version 3.0, event-based communication is the framework prime focus of interest. It defines a uniform interface for asynchronous processing of incoming events. The mental model is very similar to other popular libraries that you can find in the frontend, like — `redux-observable` or `ngrx/effects`. Both libraries had a huge influence on design and architectural decisions. 

Messaging module can be applied to variety of models, including the post popular: queue, "pub/sub"-based [microservice](../microservices/) communication, [CQRS](../cqrs.md) or EventSourcing.

{% page-ref page="../microservices/" %}

{% page-ref page="../cqrs.md" %}

{% page-ref page="../websockets.md" %}



