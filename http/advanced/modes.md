# Continuous mode

## Overview

The basic HTTP/1.1 protocol can be tricky, especially when trying to place it in frames of proper event-based thinking. In its very basic version, every request should come with a corresponding response. It is not a full-duplex communication — first a client makes the request, then a server can respond to it. That means you cannot in an easy way selectively omit/filter/combine incoming requests because every incoming event has to be consumed. The processing enforced by the protocol frames makes things very limited.

Marble.js route resolving mechanism can work in two modes — **continuous** and **non-continuous**_._ The second way is the default one which applies to 99% of possible use cases that you can model with REST APIs. Under the hood, it applies a default error handling for each incoming request making the request processing safe but tangled to disposable stream. The last 1% is for all the crazy ones. The new continuous __mode __allows you to process the stream of incoming requests in a fluent, non-detached way.

### Continuous mode

By default Marble.js HttpEffects work in non-continuous mode, which means that you have to explicitly tell the route that you would like to work in a continuous mode. In order to do that you have to append a metadata function to route builder with `continuous` flat set to true.

```typescript
  r.applyMeta({ continuous: true }),
```

The example below demonstrates an example Effect which buffers all the incoming requests till the `/flush` endpoint won't be triggered. After that all previously buffered requests will be flushed and all waiting responses will be sent back to client.

{% hint style="warning" %}
Notice that mapping to **error** and **response** comes with an initial `request` object required for sending back the response to client.
{% endhint %}

```typescript
import { useContext } from '@marblejs/core';
import { r, HttpRequestBusToken, HttpStatus } from '@marblejs/http';
import { from, of } from 'rxjs';
import { bufferWhen, catchError, filter, map, mergeMap } from 'rxjs/operators';

const getFlush$ = (req$: Observable<HttpRequest>): Observable<HttpRequest> =>
  req$.pipe(
    filter(req => req.method === 'GET' && req.url === '/flush'),
  );

const foo$ = r.pipe(
  r.applyMeta({ continuous: true }),
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect((req$, ctx) => {
    const reqBus$ = useContext(HttpRequestBusToken)(ctx.ask);
    const terminate$ = getFlush$(reqBus$);

    return req$.pipe(
      bufferWhen(() => terminate$),
      mergeMap(buffer => from(buffer)),
      mergeMap(request => processData(request).pipe(
        map(body => ({ body, request })),
        catchError(error => of({
          request,
          status: HttpStatus.BAD_REQUEST,
          body: { error: { message: error.message }}
        })),
      )),
    );
  }));
```

{% hint style="warning" %}
By treating each incoming request as an event that always has to be consumed, **continuous** mode involves the necessity of adding a custom error handling mechanism for every processed event and mapping the response with its corresponding request.
{% endhint %}

