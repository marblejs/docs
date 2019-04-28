# Available middlewares

In order to be more lightweight as possible, `@marblejs/core` package doesn't come with build-in request body parser and request logging. The core decision is to make it as more composable as possible via dedicated _Marble.js_ middleware packages:

### Request logging

{% page-ref page="logger.md" %}

### Request body parsing

{% page-ref page="body.md" %}

### Request validation

{% page-ref page="joi.md" %}

### JWT authentication

{% page-ref page="jwt/" %}

{% hint style="info" %}
All official middlewares`@marblejs/*`  are versioned in the same way as main `@marblejs/core`package.
{% endhint %}



