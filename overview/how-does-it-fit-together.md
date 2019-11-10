# How does it glue​ together?

Writing REST API in Marble.js isn't hard as you might think. The developer has to understand mostly only the main building block which is the _Effect_ and how the data flows through it.

The aim of this chapter is to demonstrate how building blocks described in previous chapters glue together. For ease of understaning lets build a tiny RESTful API for user handling. Lets omit the implementation details of database access and focus only on the Marble.js related things.

```typescript
import { createServer, combineRoutes, httpListener, r, HttpError, use } from '@marblejs/core';
import { logger$ } from '@marblejs/middleware-logger';
import { bodyParser$ } from '@marblejs/middleware-body';
import { requestValidator$, t } from '@marblejs/middleware-io';


/*------------------------
  👇 USERS API definition
-------------------------*/

const getUserValidator$ = requestValidator$({
  params: t.type({
    id: t.string,
  }),
});

const getUserList$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    mergeMapTo(getUserCollection()),
    map(body => ({ body }),
  )),
);

const getUser$ = r.pipe(
  r.matchPath('/:id'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    use(getUserValidator$),
    mergeMap(req$ => of(req.params.id).pipe(
      mergeMap(getUserById),
      map(body => ({ body }),
      catchError(() => throwError(
        new HttpError('User does not exist', HttpStatus.NOT_FOUND)
      ))
    ),
  )),
);

const users$ = combineRoutes('/users', [
  getUser$,
  getUserList$,
]);


/*------------------------
  👇 ROOT API definition
-------------------------*/

const root$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    mapTo({ body: `API version: v1` }),
  )),
);

const notFound$ = r.pipe(
  r.matchPath('*'),
  r.matchType('*'),
  r.useEffect(req$ => req$.pipe(
    mergeMapTo(throwError(
      new HttpError('Route not found', HttpStatus.NOT_FOUND)
    )),
  )),
);

const api$ = combineRoutes('/api/v1', [
  root$,
  users$,
  notFound$,
]);


/*------------------------
  👇 SERVER definition
-------------------------*/

const middlewares = [
  logger$(),
  bodyParser$(),
];

const effects = [
  api$,
];

const server = createServer({
  port: 1337,
  httpListener: httpListener({ middlewares, effects }),
});

server.run();
```

