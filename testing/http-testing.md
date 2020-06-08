---
description: '@marblejs/testing is a tool agnostic module for testing Marble.js apps.'
---

# HTTP routes testing

### Installation

To get started simply install testing module in your project as a **dev dependency.** You can use any testing framework that you like, but for example purposes will show and stick to **Jest** testing environment.

```bash
$ yarn add --dev @marblejs/testing
```

### HttpTestBed

In order to test Marble.js HTTP APIs you have to create a TestBed instance. It is a primary API with a role to configure and initialize environment for testing, overriding and injecting bound dependencies. For HTTP protocol specific use cases we will use `createHttpTestBed` factory function. In case of `HttpTestBed` type you have to provide a required HTTP listener instance and optionally default headers that will be applied for each tested request.

{% tabs %}
{% tab title="test.setup.ts" %}
```typescript
import { createHttpTestBed } from '@marblejs/testing';
import { listener } from './http.listener';

const httpTestBed = createHttpTestBed({
  listener,
  defaultHeaders: {
    ...
  },
});


// then ...


describe('user$', () => {

  test('POST "/api/v1/user" creates user', async () => {
    const dependencies = [ ... ];
    const { request, finish, ask } = await httpTestBed(dependencies);

    // 1. initialize testing environment...
    // 2. prepare and send test request...
    // 3. assert recieved response...
    // 4. ... and close connection
    
    await finish();
  });

);
```
{% endtab %}
{% endtabs %}

You can inject any bound dependencies to the TestBed constructor and ask for them eg. via `useContext` hook function.

Usually testing HTTP APIs is very repetitive - you have to create a TestBed instance, provide all basic dependencies, send a request, assert the response and finish \(close\) hanging test connection. In order to make things simple and more DRY, **@marblejs/testing** defines a useful function for defining a test setup that you can refer to in any `*.spec` file, which can save a lot of unnecessary boilerplate for defining test environment. All you have to do is to pass previously defined TestBed instance to `createTestBedSetup` function and define a default set of dependencies that later on can be overridden in test cases. Additionally you can define a set of cleanups that will be triggered on close/finish. We will go back to this topic later.

{% tabs %}
{% tab title="test.setup.ts" %}
```typescript
import { bindTo } from '@marblejs/core';
import { createHttpTestBed, createTestBedSetup } from '@marblejs/testing';
import { listener } from './http.listener';
import { UserDaoToken, UserDao } from './user.dao';
import { UserRepositoryToken, UserRepository } from './user.repository';

const testBed = createHttpTestBed({
  listener,
  defaultHeaders: {
    ...
  },
}),

export const useTestBedSetup = createTestBedSetup({
  testBed,
  dependencies: [
    bindTo(UserDaoToken)(UserDao),
    bindTo(UserRepositoryToken)(UserRepository),
    ...
  ],
  cleanups: [ ... ],
});
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="user.effect.spec.ts" %}
```typescript
import { pipe } from 'fp-ts/lib/pipeable';
import { useTestBedSetup } from './test.setup';

describe('user$', () => {
  const testBedSetup = useTestBedSetup()

  test('POST "/api/v1/user" creates user', async () => {
    const dependencies = [ ... ];
    const { request } = await testBedSetup.useTestBed();
    const body = { email: 'bob@test.com' };

    const response = await pipe(
      request('POST'),
      request.withPath('/api/v1/user'),
      request.withHeaders({ 'Authorization': 'Bearer FAKE' }),
      request.withBody(body),
      request.send,
    );

    expect(response.statusCode).toEqual(200);
    expect(response.body).toEqual(body);
  });
  
  afterEach(async () => {
    await testBedSetup.cleanup();
  });

);
```
{% endtab %}
{% endtabs %}

### Dependency injection

The main role of the TestBed, besides constructing and sending test requests, is environment setup and context preparation. You can override any binding you want and ask for bound instances in order to do some pre-test preparations. 

```typescript
import { bindTo, useContext } from '@marblejs/core';
import { pipe } from 'fp-ts/lib/pipeable';
import { useTestBedSetup } from './test.setup';
import { UserDaoToken, UserDaoMock } from './user.dao';
import { UserRepositoryToken, UserRepository } from './user.repository';

describe('user$', () => {
  const testBedSetup = useTestBedSetup()

  test('POST "/api/v1/user" creates user', async () => {
  
    // override binding
    const dependencies = [
      bindTo(UserDaoToken)(UserDaoMock),
    ];
    
    const { request, ask } = await testBedSetup.useTestBed(dependencies);
    const body = { email: 'bob@test.com' };
    
    // access bound instances 
    const userRepository = useContext(UserRepositoryToken)(ask);
    ...

    const response = await pipe(
      request('POST'),
      request.withPath('/api/v1/user'),
      request.withHeaders({ 'Authorization': 'Bearer FAKE' }),
      request.withBody(body),
      request.send,
    );

    expect(response.statusCode).toEqual(200);
    expect(response.body).toEqual(body);
  });
  
  afterEach(async () => {
    await testBedSetup.cleanup();
  });

);
```

### Constructing test request

Marble.js defines similar pipeable API as for constructing HTTP routes. It defines a set of useful functions for building and sending a request in a functional way.

```
request :: HttpMethod -> HttpTestBedRequest
```

```
withPath :: string -> HttpTestBedRequest -> HttpTestBedRequest
```

```haskell
withHeaders :: HttpHeaders -> HttpTestBedRequest -> HttpTestBedRequest
```

```haskell
withBody :: Body -> HttpTestBedRequest -> HttpTestBedRequest & WithBodyApplied
```

```haskell
send :: HttpTestBedRequest -> Promise HttpTestBedRequest
```

```typescript
// const { request } = await testBedSetup.useTestBed();
// const { request } = await httpTestBed();
    
const response = await pipe(
  request('POST'),
  request.withPath('/api/v1/user'),
  request.withHeaders({ 'Authorization': 'Bearer FAKE_TOKEN' }),
  request.withBody(data),
  request.send,
);

typeof response === HttpTestBedResponse {
  req: HttpTestBedRequest<any>;
  metadata: TestingMetadata;
  statusCode: HttpStatus;
  statusMessage?: string;
  headers: HttpHeaders;
  body?: any;
}
```

### Cleanups

Sometimes there is a need to do some cleanup stuff after each test. Of course, you can inject and close all hanging connections manually for each dependency that requires it, but as you probably know... it is not a best idea. Marble.js cannot automagically close all established connections that bound dependencies created, but you can instruct TestBedSetup by defining a set of cleanups to trigger a desired job after finish.

Let's assume that we have some custom dependency that their interface defines an async `close` method.

{% tabs %}
{% tab title="custom.reader.ts" %}
```typescript
interface CustomDependency {
  ...
  close: () => Promise<void>,
};

export const CustomDependency = createReader(_ => ... );

export const CustomDependencyToken = createContextToken<CustomDependency>('CustomDependency');
```
{% endtab %}
{% endtabs %}

In order to create a `DependencyCleanup` you have to provide a lookup \(context\) token and implement a cleanup method that will return a promise.

```typescript
import { bindTo } from '@marblejs/testing';
import { DependencyCleanup, createTestBedSetup } from '@marblejs/testing';
import { listener } from './http.listener';
import { CustomDependency, CustomDependencyToken } from './custom.reader';

const customDependencyCleanup: DependencyCleanup<CustomDependency> = {
  token: CustomDependencyToken,
  cleanup: dep => dep.close(),
};

const testBed = createHttpTestBed({ listener }),

export const useTestBedSetup = createTestBedSetup({
  testBed,
  dependencies: [
    bindTo(CustomDependencyToken)(CustomDependency),
    ...
  ],
  cleanups: [
    customDependencyCleanup,
    ...
  ],
});
```

