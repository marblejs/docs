# FAQ

## Does Marble.js uses Express underneath?

No! Express is a cool library but it shows its age. From the very beginning the aim of Marble.js was to build it from scratch with its own philosophy in mind. Thanks to the design decissions that were set at the beginning of its existence, Marble.js is faster than Express and comparable to the most performant HTTP libraries available on the Node.js platform.

## Why the heck I need here RxJS? 

Despite the single event nature of basic HTTP, there are no contradictions against using it for single events. In Marble, RxJS is used as a hammer for expressing asynchronous flow with monadic manner, even if you have to deal with only one event passing over time. Marble.js doesn't operate only over basic [HTTP](../overview/) protocol but can be used also for both [WebSocket](../websockets/) and event sourcing purposes, where the multi-event nature fits best. Don't be scared of the complexity and abstractions presented in RxJS API — the Marble.js framework, in general, is incredibly simple.



