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

The bootstrapping consists of two very simple steps: _HTTP handler_ definition and _HTTP server_ configuration.

`httpListener` is the starting point of every _Marble.js_ application. It includes definitions of all _middlewares_ and API _effects_.

{% code title="app.ts" %}
```typescript
const middlewares = [
  logger$,
  bodyParser$,
];

const effects = [
  endpoint1$,
  endpoint2$,
  ...
];

export const app = httpListener({ middlewares, effects });
```
{% endcode %}

Because ****_Marble.js_ is built on top of [_Node.js_](https://nodejs.org/en/) __platform and doesn't create any abstractions for server bootstrapping - all you need to do is to call `createServer` with initialized _app_ and then start listening to given _port_ and _hostname_.

{% code title="server.ts" %}
```typescript
import { app } from './app.ts';

const httpServer = http
  .createServer(app)
  .listen(PORT, HOSTNAME);
```
{% endcode %}

