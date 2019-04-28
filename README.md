# Introduction

![](.gitbook/assets/logo-0-5x.png)

**Marble.js** is a functional reactive HTTP middleware framework built on top of [Node.js](http://nodejs.org) platform, [TypeScript](https://www.typescriptlang.org) and [RxJS](http://reactivex.io/rxjs) library.

> _If you don't have any experience with functional reactive programming, we strongly recommend to gain some basic overview first with_ [_ReactiveX intro_](http://reactivex.io/intro.html) _or with_ [_The introduction to Reactive Programming you've been missing_](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754) _written by_ [_@andrestaltz_](https://twitter.com/andrestaltz)_._

_**See our GitHub page**:_ [_github.com/marblejs/marble_](https://github.com/marblejs/marble)

## Philosophy

If we think closely how typical HTTP API works we can quickly recognize that it deals with streams of asynchonous events also called as HTTP requests. Describing it very briefly - typically each request needs to be transformed into response that goes back to the client \(which is our event initiator\) using custom middlewares or designated endpoints. In reactive programming world, all those core concepts we can translate into very simple marble diagram:

![](.gitbook/assets/flow%20%281%29.png)

In this world everyting is a stream. The core concept of **Marble.js** is based on the event flow of marble diagrams which can be used to visually express time based behaviour of HTTP streams. Ok, but why the heck we need those `Observables`? Trends come and go, but asynchronously nature of JavaScript and Node.js platform constantly evolves. With reactive manner we can deliver complex features faster by providing the ability to compose complex tasks with ease and with less amount of code. If you have ever worked with libraries like [Redux Observable](https://redux-observable.js.org), [@ngrx/effects](https://github.com/ngrx/platform/blob/master/docs/effects/README.md) or other libraries that leverages functional reactive paradigm, you will feel like in home. Still there? So lets get started!

## Installation

**Marble.js** requires node **v8.0** or higher:

```bash
$ npm i @marblejs/core rxjs
```

or if you are a hipster:

```bash
$ yarn add @marblejs/core rxjs
```



