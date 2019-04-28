# Getting started

## Installation

**Marble.js** requires node **v8.0** or higher:

```bash
$ npm i @marblejs/core rxjs
```

or if you are a hipster:

```bash
$ yarn add @marblejs/core rxjs
```

## Bootstrapping

The very basic configuration consists of two steps: _HTTP listener_ definition and _HTTP server_ configuration.

`httpListener` is the basic starting point of every Marble application. It includes definitions of all _middlewares_ and API _effects_.

{% code-tabs %}
{% code-tabs-item title="http.listener.ts" %}
```typescript
const middlewares = [
  logger$(),
  bodyParser$(),
  // ...
];

const effects = [
  endpoint1$,
  endpoint2$,
  // ...
];

export default httpListener({ middlewares, effects });
```
{% endcode-tabs-item %}
{% endcode-tabs %}

To create Marble app instance, we can use [`createServer`](../api-reference/core/createserver.md), which is a wrapper around Node.js server creator with much more possibilities and goods inside. When created, it won't automatically start listening to given port and hostname until you call `.run()` method on it.

{% code-tabs %}
{% code-tabs-item title="server.ts" %}
```typescript
import { createServer } from '@marblejs/core';
import httpListener from './http.listener';

export const server = createServer({
  port: 1337,
  hostname: '127.0.0.1',
  httpListener,
});

server.run();
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
For more API specific details about server bootstraping, visit [createServer](../api-reference/core/createserver.md) API reference.
{% endhint %}

If you prefer to have standard Node.js control over server creation, you can also easily hook the app directly into Node.js `http.createServer`

{% code-tabs %}
{% code-tabs-item title="server.ts" %}
```typescript
import { createContext } from '@marblejs/core';
import * as http from 'http';
import httpListener from './http.listener';

const httpServer = httpListener
  .run(createContext());
  
export const server = http
  .createServer(httpServer)
  .listen(1337, '127.0.0.1');
```
{% endcode-tabs-item %}
{% endcode-tabs %}

In the next chapter you will learn how to create basic **Marble.js** endpoints using [Effects](effects.md).

