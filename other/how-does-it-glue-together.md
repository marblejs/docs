# How does it glue together?

Writing REST API in Marble.js isn't hard as you might think. The developer has to understand mostly only the main building block which is the Effect and how the data flows through it.

The aim of this chapter is to demonstrate how building blocks described in previous chapters glue together. For ease of understanding, lets build a tiny RESTful API for user handling. Lets omit the implementation details of database access and focus only on the Marble.js related things.

```typescript
import { createServer, combineRoutes, httpListener, r, HttpError, HttpStatus } from '@marblejs/http';
import { logger$ } from '@marblejs/middleware-logger';
import { bodyParser$ } from '@marblejs/middleware-body';
import { requestValidator$, t } from '@marblejs/middleware-io';
import { catchError, mergeMapTo, mergeMap, mapTo, map } from 'rxjs/operators';
import { of, throwError, from } from 'rxjs';
import { IO } from 'fp-ts/lib/IO';
import { pipe } from 'fp-ts/lib/function';

/*------------------------
  👇 utility functions
-------------------------*/

const getUserCollection = () =>
  from([{ id: '1' }]);

const getUserById = (id: string) =>
  id === '1'
    ? of({ id: '1', name: 'Test' })
    : throwError(new Error('User not found'));

/*------------------------
  👇 USERS API definition
-------------------------*/

const validateRequest = requestValidator$({
  params: t.type({
    id: t.string,
  }),
});

const getUserList$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    mergeMap(getUserCollection),
    map(body => ({ body })),
  )),
);

const getUser$ = r.pipe(
  r.matchPath('/:id'),
  r.matchType('GET'),
  r.useEffect(req$ => req$.pipe(
    validateRequest,
    mergeMap(req => pipe(
      getUserById(req.params.id),
      catchError(() => throwError(() =>
        new HttpError('User does not exist', HttpStatus.NOT_FOUND)
      ))
    )),
    map(body => ({ body })),
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

const api$ = combineRoutes('/api/v1', [
  root$,
  users$,
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
  listener: httpListener({ middlewares, effects }),
});

const main: IO<void> = async () =>
  await (await server)();

main();
```

