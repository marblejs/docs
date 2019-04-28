# Introduction

![](.gitbook/assets/logo-0-5x.png)

**Marble.js** is a functional reactive [Node.js](http://nodejs.org) framework for building **server-side** applications, based on [TypeScript](https://www.typescriptlang.org) and [RxJS](http://reactivex.io/rxjs).

## Philosophy

The core concept of **Marble.js** assumes that almost everything is a stream. The main building block of the whole framework is an _Effect_, which is just a function that returns a stream of events.

The purely functional languages like [Haskell](https://en.wikipedia.org/wiki/Haskell_%28programming_language%29) express side effects such as IO and other stateful computations using monadic actions. With the big popularity of  [RxJS](http://rxjs.dev) _Observable_ monad, you can create a referential transparent program specification made up of functions that may produce side effects like network, logging, database access, etc. Using its monadic nature we can map I/O operations over effects and flat them to bring in other __sequences of operations. Marble.js is a functional reactive framework, that's why _RxJS_ is a first class citizen here. It’s a much more powerful and feature-rich monad than _IO_ that implements the basic abstract interface as well as a ton of additional functionalities for manipulating sequences of events over time.

When looking at Marble.js you can ask: **"Why do we need RxJS for HTTP?"**. Despite the single event nature of basic HTTP, there are no contradictions against using it for single events. In Marble, _RxJS_ is used as a hammer for expressing asynchronous flow with monadic manner, even if you have to deal with only one event passing over time. Marble.js doesn't operate only over basic [HTTP](overview/) protocol but can be used also for both [WebSocket](websockets/) and event sourcing purposes, where the multi-event nature fits best. Don't be scared of the complexity and abstractions presented in _RxJS_ API —  the Marble.js framework, in general, is incredibly simple. For more details about its specifics, please visit the next chapters that will guide you through the framework environment and implementation details.

_For those who are curious about the framework name - it comes from a popular way of visually expressing the time-based behavior of event streams, aka marble diagrams. This kind of domain-specific language is a popular way of testing asynchronous streams, especially in RxJS environments._

{% hint style="success" %}
👉 If you have ever worked with libraries like [Redux Observable](https://redux-observable.js.org), [@ngrx/effects](https://github.com/ngrx/platform/blob/master/docs/effects/README.md) or other libraries that leverage functional reactive paradigm, you will feel like in home.
{% endhint %}

{% hint style="info" %}
👉 If you don't have any experience with functional reactive programming, we strongly recommend to gain some basic overview first with [ReactiveX intro](http://reactivex.io/intro.html) or with [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754) written by [@andrestaltz](https://twitter.com/andrestaltz).
{% endhint %}

## Installation

**Marble.js** requires node **v8.0** or higher:

```bash
$ npm i @marblejs/core rxjs
```

or if you are a hipster:

```bash
$ yarn add @marblejs/core rxjs
```

## Examples

To view the example project, visit the [example](https://github.com/marblejs/example) repository.

