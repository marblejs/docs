---
description: HttpEffect route builder based on IxMonad
---

# r.pipe

## **Importing**

```typescript
import { r } from '@marblejs/http';
```

## `pipe`

`r` namespace function. Creates pipeable _RouteEffect_ builder.

### **Type declaration**

```typescript
pipe :: ...Arity<IxBuilder, IxBuilder> -> RouteEffect
```

{% hint style="warning" %}
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

1. **`<optional> applyMeta`**
2. **`matchPath`**
3. **`matchType`**
4. **`use`** \[...\]
5. **`useEffect`**
6. **`applyMeta`** \[...\]

## **`matchPath`**

`r` namespace function. Matches request path for connected _HttpEffect_.

### **Type declaration**

```typescript
matchPath :: string -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `string` |

## **`matchType`**

`r` namespace function. Matches HTTP method type for connected _HttpEffect_.

### **Type declaration**

```typescript
matchType :: HttpMethod -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _path_ | `HttpMethod` |

## `use`

`r` namespace function. Registers HTTP middleware with connected _HttpEffect._

### **Type declaration**

```typescript
use :: HttpMiddlewareEffect -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _middleware_ | `HttpMiddlewareEffect` |

## `useEffect`

`r` namespace function. Registers _HttpEffect._

### **Type declaration**

```typescript
useEffect :: HttpEffect -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| _effect_ | `HttpEffect` |

## `applyMeta`

`r` namespace function. Applies metadata to connected _HttpEffect_.

### **Type declaration**

```typescript
applyMeta :: RouteMeta -> IxBuilder -> IxBuilder
```

### **Parameters**

| _parameter_ | definition |
| :--- | :--- |
| meta | `RouteMeta` |

`RouteMeta` attributes:

| parameter | type | definition |
| :--- | :--- | :--- |
| `name` | &lt;optional&gt; `string` | route/effect name |
| `continuous` | &lt;optional&gt; `boolean` | enables [continuous mode](../../../http/advanced/modes.md) |
| `overridable` | &lt;optional&gt; `boolean` | if true, the route can be overrode by another route |

## Example

```typescript
import { r } from '@marblejs/http';

const example$ = r.pipe(
  r.applyMeta({ continuous: true }),
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

