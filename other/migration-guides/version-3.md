---
description: >-
  This chapter provides a set of guidelines to help you migrate from Marble.js
  version 3.x to the latest 4.x version.
---

# Migration from version 3.x

The newest iteration comes with some API breaking change, but don’t worry, these are not game-changers, but rather convenient improvements that open the doors to new future possibilities. During the development process, the goal was to notify and discuss incoming changes within the community as early as possible. You can check here [what has changed since the latest version](https://github.com/marblejs/marble/issues/172).

{% hint style="info" %}
For more detailed info about more low-level improvements, please visit release candidate [changelog](https://github.com/marblejs/marble/releases/tag/v4.0.0-rc.1).
{% endhint %}

## @marblejs/core

{% hint style="danger" %}
Legacy `EffectFactory` builder is no longer available. Please user `r` builder instead.
{% endhint %}

{% hint style="danger" %}
Legacy `switchToProtocol` operator is no longer available.
{% endhint %}

## @marblejs/http

Moved HTTP-related API's from `@marblejs/core` to new package `@marblejs/http`, eg.

**❌ Old \(example\):**

```typescript
import { HttpRequest, HttpEffect, useContext } from '@marblejs/core';
```

**✅ New \(example\):**

```typescript
import { useContext } from '@marblejs/core';
import { HttpRequest, HttpEffect } from '@marblejs/http';
```

**HttpOutputEffect -** Improved interface

**❌ Old:**

```typescript
Observable<{ req, res: { status, body, headers } }>
  -> Observable<{ status, body, headers }>
```

**✅ New:**

```typescript
Observable<{ status, body, headers, request }>
  -> Observable<{ status, body, headers, request }>
```

**HttpErrorEffect -** Improved interface

**❌ Old:**

```typescript
Observable<{ req, error }>
  -> Observable<{ status, body, headers }>
```

**✅ New:**

```typescript
Observable<{ error, request }>
  -> Observable<{ status, body, headers, request }>
```

## @marblejs/messaging

**❌ Old:**

```typescript
import { messagingClient, eventBus, eventBusClient } from '@marblejs/messaging';
```

**✅ New:**

```typescript
import { MessagingClient, EventBus, EventBusClient } from '@marblejs/messaging';
```

## @marblejs/websockets

Version 4.x removes deprecated legacy `connection$` attribute. Please use HTTP `upgrade` event for checking connections.

## @marblejs-contrib/middleware-joi

**❌ Old:**

```text
npm i @marblejs/middleware-joi
```

**✅ New:**

```text
npm i @marblejs-contrib/middleware-joi
```

## @marblejs-contrib/middleware-jwt

**❌ Old:**

```text
npm i @marblejs/middleware-jwt
```

**✅ New:**

```text
npm i @marblejs-contrib/middleware-jwt
```

