# Installation

**Marble.js** requires node **v8.0** or higher. In case you would like to work with `@marblejs/http` module:

```bash
$ npm i @marblejs/core @marblejs/http fp-ts rxjs
```

or if you are a hipster:

```bash
$ yarn add @marblejs/core @marblejs/http fp-ts rxjs
```

Every `@marblejs/*` package requires `@marblejs/core` + `fp-ts` + `rxjs` to be installed first.

| package | description |
| :--- | :--- |
| [@marblejs/core](../other/api-reference/core/) | Core module |
| [@marblejs/http](../other/api-reference/marblejs-http/) | HTTP module |
| [@marblejs/messaging](../other/api-reference/messaging/) | Messaging module |
| [@marblejs/websockets](../other/api-reference/websockets/) | WebSocket module |
| [@marblejs/testing](../testing/http-testing.md) | Testing module |
| [@marblejs/middleware-logger](../other/api-reference/middleware-logger.md) | Logger middleware |
| [@marblejs/middleware-body](../other/api-reference/middleware-body.md) | Body parser middleware |
| [@marblejs/middleware-io](../other/api-reference/middleware-io.md) | I/O validation middleware |
| [@marblejs/middleware-jwt](../other/api-reference/middleware-jwt/) | JWT authorization middleware |
| [@marblejs/middleware-cors](../other/api-reference/middleware-cors.md) | CORS middleware |
| [@marblejs/middleware-multipart](../other/api-reference/middleware-multipart.md) | Multipart middleware |

