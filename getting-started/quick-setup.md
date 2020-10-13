# Quick setup

The very basic configuration consists of two steps: HTTP listener definition and HTTP server configuration.

`httpListener` is the basic starting point of every Marble application. It includes definitions of all global middlewares and API effects.

{% tabs %}
{% tab title="http.listener.ts" %}
```typescript
import { httpListener } from '@marblejs/core';
import { logger$ } from '@marblejs/middleware-logger';
import { bodyParser$ } from '@marblejs/middleware-body';
import { api$ } from './api.effects';

const middlewares = [
  logger$(),
  bodyParser$(),
  // middleware3$
  // middleware4$
  // ...
];

const effects = [
  api$,
  // endpoint2$
  // endpoint3$
  // ...
];

export const listener = httpListener({
  middlewares,
  effects,
});
```
{% endtab %}
{% endtabs %}

And here is our simple "hello world" endpoint.

{% tabs %}
{% tab title="api.effects.ts" %}
```typescript
import { r } from '@marblejs/core';
import { mapTo } from 'rxjs/operators';

export const api$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
     mapTo({ body: 'Hello, world!' }),
  )));
```
{% endtab %}
{% endtabs %}

To create Marble app instance, we can use [`createServer`](../other/api-reference/core/createserver.md), which is a wrapper around Node.js server creator with much more possibilities and goods inside. When created, it won't automatically start listening to given port and hostname until you call its awaited instance.

{% tabs %}
{% tab title="index.ts" %}
```typescript
import { createServer } from '@marblejs/core';
import { IO } from 'fp-ts/lib/IO';
import { listener } from './http.listener';

const server = createServer({
  port: 1337,
  hostname: '127.0.0.1',
  listener,
});

const main: IO<void> = async () =>
  await (await server)();

main();
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To see Marble.js in action you can visit [example repository](https://github.com/marblejs/example) for a complete Marble.js app example.
{% endhint %}

We'll use [TypeScript](https://www.typescriptlang.org/) in the documentation but you can always write Marble apps in standard JavaScript \(and any other language that transpiles to JavaScript\).

To test run your server you can install `typescript` compiler and `ts-node`:

```bash
$ yarn add typescript ts-node
```

Add the following script to your `package.json` file:

```javascript
"scripts": {
  "start": "ts-node index.ts"
}
```

Now go ahead, create `index.ts`, `http.listener.ts`, `api.effects.ts` modules in your project and run your server:

```bash
$ yarn start
```

Finally test your "functional" server by visiting [http://localhost:1337](quick-setup.md)

{% hint style="info" %}
For more API specific details about server bootstrapping, visit [createServer](../other/api-reference/core/createserver.md) API reference
{% endhint %}

In the next HTTP chapter you will learn how to create basic **Marble.js** endpoints using [Effects](quick-setup.md), how to build and compose middlewares and how to build a basic REST API routing.

