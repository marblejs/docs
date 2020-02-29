---
description: >-
  Combines routing for different Effects, prefixed with path passed as a first
  argument.
---

# combineRoutes

### **Importing**

```typescript
import { combineRoutes } from '@marblejs/core';
```

### **Type declaration**

```text
combineRoutes :: (string, RouteCombinerConfig) -> RouteEffectGroup
combineRoutes :: (string, (RouteEffect | RouteEffectGroup)[]) -> RouteEffectGroup
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `string` |
| _configOrEffects_ | `RouteCombinerConfig | Array<RouteEffect | RouteEffectGroup>` |

#### _**RouteCombinerConfig**_

| _parameter_ | definition |
| :--- | :--- |
| effects | `Array<RouteEffect | RouteEffectGroup>` |
| middlewares | &lt;optional&gt; `Array<HttpMiddlewareEffect>` |

### Returns

Factorized `RouteEffectGroup` object.

### Example

{% code title="user.effects.ts" %}
```typescript
import { combineRoutes } from '@marblejs/core';
import { authorize$ } from 'auth.middleware';

// const getUsers$ = ...
// const postUser$ = ...

export const user$ = combineRoutes('/user', {
  middlewares: [authorize$],
  effects: [getUsers$, postUser$],
});
```
{% endcode %}

{% code title="api.effects.ts" %}
```typescript
import { combineRoutes } from '@marblejs/core';
import { user$ } from './user';

// const root$ = ...
// const notFound$ = ...

export const api$ = combineRoutes('/api/v1', [
  root$, user$, notFound$,
]);
```
{% endcode %}

