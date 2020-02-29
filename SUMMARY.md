# Table of contents

* [Marble.js](README.md)

## Getting started

* [Installation](getting-started/installation.md)
* [Quick setup](getting-started/quick-setup.md)

## HTTP

* [Effects](http/effects.md)
* [Middlewares](http/middlewares.md)
* [Routing](http/routing.md)
* [Errors](http/errors.md)
* [Output](http/output.md)
* [Context](http/context.md)
* [Advanced](http/advanced/README.md)
  * [Logging](http/advanced/logging.md)
  * [Validation](http/advanced/validation.md)
  * [Server Events](http/advanced/server-events.md)
  * [Streaming](http/advanced/streaming.md)
  * [Continuous mode](http/advanced/modes.md)

## Messaging

* [Core concepts](messaging/core-concepts/README.md)
  * [Events](messaging/core-concepts/events.md)
  * [Effects](messaging/core-concepts/effects.md)
* [Microservices](messaging/microservices/README.md)
  * [AMQP \(RabbitMQ\)](messaging/microservices/rabbitmq.md)
  * [Redis Pub/Sub](messaging/microservices/redis-pub-sub.md)
* [CQRS](messaging/cqrs.md)
* [WebSockets](messaging/websockets.md)

## Other

* [How does it glue together?](other/how-does-it-glue-together.md)
* [Migration guides](other/migration-guides/README.md)
  * [Migration from version 2.x](other/migration-guides/version-2.md)
  * [Migration from version 1.x](other/migration-guides/version-1.md)
* [API reference](other/api-reference/README.md)
  * [core](other/api-reference/core/README.md)
    * [bindTo](other/api-reference/core/bind.md)
    * [bindEagerlyTo](other/api-reference/core/bindeagerlyto.md)
    * [createEvent](other/api-reference/core/createevent.md)
    * [createServer](other/api-reference/core/createserver.md)
    * [combineRoutes](other/api-reference/core/core-combineroutes.md)
    * [createContextToken](other/api-reference/core/createinjectiontoken.md)
    * [EffectFactory](other/api-reference/core/core-effectfactory.md)
    * [r.pipe](other/api-reference/core/r.pipe.md)
    * [httpListener](other/api-reference/core/core-httplistener.md)
    * [operator: matchEvent](other/api-reference/core/operator-matchevent.md)
    * [operator: use](other/api-reference/core/operator-use.md)
    * [operator: act](other/api-reference/core/operator-act.md)
  * [messaging](other/api-reference/messaging/README.md)
    * [reply](other/api-reference/messaging/reply.md)
  * [websockets](other/api-reference/websockets/README.md)
    * [webSocketListener](other/api-reference/websockets/websocketlistener.md)
    * [operator: broadcast](other/api-reference/websockets/operator-broadcast.md)
    * [operator: mapToServer](other/api-reference/websockets/operator-maptoserver.md)
  * [middleware-multipart](other/api-reference/middleware-multipart.md)
  * [middleware-cors](other/api-reference/middleware-cors.md)
  * [middleware-joi](other/api-reference/middleware-joi.md)
  * [middleware-jwt](other/api-reference/middleware-jwt/README.md)
    * [Token signing](other/api-reference/middleware-jwt/token-creation.md)
  * [middleware-io](other/api-reference/middleware-io.md)
  * [middleware-logger](other/api-reference/middleware-logger.md)
  * [middleware-body](other/api-reference/middleware-body.md)
* [Style Guide](other/style-guide.md)
* [FAQ](other/faq.md)

