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

## Testing

* [HTTP routes testing](testing/http-testing.md)

## Other

* [How does it glue together?](other/how-does-it-glue-together.md)
* [Migration guides](other/migration-guides/README.md)
  * [Migration from version 2.x](other/migration-guides/version-2.md)
  * [Migration from version 1.x](other/migration-guides/version-1.md)
* [API reference](other/api-reference/README.md)
  * [@marblejs/core](other/api-reference/core/README.md)
    * [bindTo](other/api-reference/core/bind.md)
    * [bindEagerlyTo](other/api-reference/core/bindeagerlyto.md)
    * [createEvent](other/api-reference/core/createevent.md)
    * [createContextToken](other/api-reference/core/createinjectiontoken.md)
    * [operator: matchEvent](other/api-reference/core/operator-matchevent.md)
    * [operator: use](other/api-reference/core/operator-use.md)
    * [operator: act](other/api-reference/core/operator-act.md)
  * [@marblejs/http](other/api-reference/marblejs-http/README.md)
    * [httpListener](other/api-reference/marblejs-http/core-httplistener.md)
    * [r.pipe](other/api-reference/marblejs-http/r.pipe.md)
    * [combineRoutes](other/api-reference/marblejs-http/core-combineroutes.md)
    * [createServer](other/api-reference/marblejs-http/createserver.md)
  * [@marblejs/messaging](other/api-reference/messaging/README.md)
    * [eventBus](other/api-reference/messaging/eventbus.md)
    * [messagingClient](other/api-reference/messaging/messagingclient.md)
    * [createMicroservice](other/api-reference/messaging/createmicroservice.md)
    * [reply](other/api-reference/messaging/reply.md)
  * [@marblejs/websockets](other/api-reference/websockets/README.md)
    * [webSocketListener](other/api-reference/websockets/websocketlistener.md)
    * [operator: broadcast](other/api-reference/websockets/operator-broadcast.md)
    * [operator: mapToServer](other/api-reference/websockets/operator-maptoserver.md)
  * [@marblejs/middleware-multipart](other/api-reference/middleware-multipart.md)
  * [@marblejs/middleware-cors](other/api-reference/middleware-cors.md)
  * [@marblejs/middleware-io](other/api-reference/middleware-io.md)
  * [@marblejs/middleware-logger](other/api-reference/middleware-logger.md)
  * [@marblejs/middleware-body](other/api-reference/middleware-body.md)
  * [@marblejs-contrib/middleware-jwt](other/api-reference/middleware-jwt/README.md)
    * [Token signing](other/api-reference/middleware-jwt/token-creation.md)
  * [@marblejs-contrib/middleware-joi](other/api-reference/middleware-joi.md)
* [Style Guide](other/style-guide.md)
* [FAQ](other/faq.md)

