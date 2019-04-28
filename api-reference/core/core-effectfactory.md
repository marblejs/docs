---
description: Set of factory functions for building router Effect.
---

# EffectFactory

## **Importing**

```typescript
import { EffectFactory } from '@marblejs/core';
```

## + **matchPath**

`EffectFactory` namespace function. Matches request path for connected _Effect_.

### **Type declaration**

```typescript
matchPath :: string -> matchType
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `string` |

### Returns

EffectFactory __`matchType` function

## + **matchType**

`EffectFactory` namespace function. Matches HTTP method type for connected _Effect_.

### **Type declaration**

```typescript
matchType :: HttpMethod -> use
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _type_ | `HttpMethod = 'POST | 'PUT' | 'PATCH' | 'GET' | 'HEAD' | 'DELETE' | 'CONNECT' | 'OPTIONS' | 'TRACE' | '*'` |

### Returns

EffectFactory _`use`_ function

## + **use**

`EffectFactory` namespace function. Connects _Effect_ with path and HTTP method type.

### **Type declaration**

```typescript
use :: HttpEffect -> RouteEffect
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _effect_ | `HttpEffect` function |

### Returns

Factorized `RouteEffect` object.

## Example

{% code-tabs %}
{% code-tabs-item title="root.effect.ts" %}
```typescript
import { EffectFactory } from '@marblejs/core';

export const root$ = EffectFactory
  .matchPath('/')
  .matchType('GET')
  .use(req$ => req$.pipe(
    mapTo({ body: `Hello, world! 👻` })
  ));
```
{% endcode-tabs-item %}
{% endcode-tabs %}



