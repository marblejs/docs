---
description: HttpEffect route builder based on IxMonad
---

# r.pipe

## **Importing**

```typescript
import { r } from '@marblejs/core';
```

## + pipe

`r` namespace function. Creates pipeable _RouteEffect_ builder.

### **Type declaration**

```typescript
pipe :: ...Arity<IxBuilder, IxBuilder> -> RouteEffect
```

{% hint style="danger" %}
`r.pipe` builder pays attention to the order of applied operators.
{% endhint %}

```typescript
const example$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.use(middleware_1$),
  r.useEffect(req$ => req$.pipe(
    // ...
  )),
  r.use(middleware_2$), // ❌ type error!
);

// or 

const example$ = r.pipe(
  r.matchType('GET'),
  r.matchPath('/'), // ❌ type error!
  r.use(middleware_1$),
  r.use(middleware_2$), 
  r.useEffect(req$ => req$.pipe(
    // ...
  )),
);
```

**Correct order:**

1. **`matchPath`**
2. **`matchType`**
3. **`use`** ...
4. **`useEffect`**
5. **`applyMeta` ...**

## + **matchPath**

`r` namespace function. Matches request path for connected _HttpEffect_.

### **Type declaration**

```typescript
matchPath :: string -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `string` |

## + **matchType**

`r` namespace function. Matches HTTP method type for connected _HttpEffect_.

### **Type declaration**

```typescript
matchType :: HttpMethod -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `HttpMethod` |

## + use

`r` namespace function. Registers HTTP middleware with connected _HttpEffect._

### **Type declaration**

```typescript
use :: HttpMiddlewareEffect -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _middleware_ | `HttpMiddlewareEffect` |

## + useEffect

`r` namespace function. Registers _HttpEffect._

### **Type declaration**

```typescript
useEffect :: HttpEffect -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _effect_ | `HttpEffect` |

## + applyMeta

`r` namespace function. Applies metadata to connected _HttpEffect_.

### **Type declaration**

```typescript
applyMeta :: Record<string, any> -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| meta | `Record<string, any>` |

## Example

```typescript
import { r } from '@marblejs/core';

const example$ = r.pipe(
  r.matchPath('/'),
  r.matchType('GET'),
  r.use(middleware_1$),
  r.use(middleware_2$),
  r.useEffect(req$ => req$.pipe(
    // ...
  )),
  r.applyMeta({ meta_1: /* ... */ }),
  r.applyMeta({ meta_1: /* ... */ }),
);
```

