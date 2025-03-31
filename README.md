# f-promise-async

Originally forked from [f-promise](https://github.com/Sage/f-promise), that was a tool managing legacy Promise-oriented coroutines for node.js.

Contrary to `f-promise`, `f-promise-async` does not allow to manage promises, but only provides some tools to help promise manipulation and synchronization.

## Installation

```sh
npm install f-promise-async
```

## TypeScript support

TypeScript is fully supported.

## Callbacks support

You can also use `f-promise-async` with callback APIs. 
So you don't absolutely need wrappers like `mz/fs`, you can directly call node's `fs` API:

```javascript
import { wait } from 'f-promise-async';

// callback style
import * as fs from 'fs';
const directories = await wait(cb => fs.readdir(path, cb));
````

## Control Flow utilities

These goodies solve some common problems.

### run/wait

* `promise = run(() => { wait(promise/callback); ... })`  
  create a fake coroutine to write asynchronous code in a asynchronous way !  
  keep that methode to ease f-promise conversion to async/await  
  but also for global context
  * `promise = run(fn)` create a fake coroutine.
  Contrary to legacy `f-promise`, you can use other tools without `run`, except Continuation local storage.
  * `result = await wait(promise/callback)` encapsulate callback.

### funnel

* `fun = funnel(max)`  
  limits the number of concurrent executions of a given code block.  
  The `funnel` function is typically used with the following pattern:

```ts  
// somewhere  
var myFunnel = funnel(10); // create a funnel that only allows 10 concurrent executions.  
// elsewhere  
myFunnel(async function() { /* code with at most 10 concurrent executions */ });  
```  

The `funnel` function can also be used to implement critical sections. Just set funnel's `max` parameter to 1.  
If `max` is set to 0, a default number of parallel executions is allowed.  
This default number can be read and set via `flows.funnel.defaultSize`.  
If `max` is negative, the funnel does not limit the level of parallelism.  
The funnel can be closed with `fun.close()`.  
When a funnel is closed, the operations that are still in the funnel will continue but their callbacks 
won't be called, and no other operation will enter the funnel.

### handshake and queue

* `hs = handshake()`  
  allocates a simple semaphore that can be used to do simple handshakes between two tasks.  
  The returned handshake object has two methods:  
  `await hs.wait()`: waits until `hs` is notified.  
  `hs.notify()`: notifies `hs`.  
  Note: `wait` calls are not queued. An exception is thrown if wait is called while another `wait` is pending.

* `q = new Queue(options)`  
  allocates a queue which may be used to send data asynchronously between two tasks.  
  The `max` option can be set to control the maximum queue length.  
  When `max` has been reached `q.put(data)` discards data and returns false.  
  The returned queue has the following methods:  
  `data = await q.read()`:  dequeue and returns the first item. Waits if the queue is empty. Does not allow concurrent read.  
  `await q.write(data)`:  queues an item. Waits if the queue is full.  
  `ok = q.put(data)`: queues an item synchronously. Returns true if the queue accepted it, false otherwise.  
  `q.end()`: ends the queue. This is the synchronous equivalent of `q.write(_, undefined)`  
  `data = q.peek()`: returns the first item, without dequeuing it. Returns `undefined` if the queue is empty.  
  `array = q.contents()`: returns a copy of the queue's contents.  
  `q.adjust(fn[, thisObj])`: adjusts the contents of the queue by calling `newContents = fn(oldContents)`.

### Continuation local storage (CLS)

* `result = await withContext(fn, cx)`  
  wraps a function so that it executes with context `cx` (or a wrapper around current context if `cx` is falsy).  
  The previous context will be restored when the function returns (or throws).  
  returns the wrapped function.  
  It is the only tool that need to be executed in a `run`. Can be a nested function call (coroutine).

### Miscellaneous

* [sleep](./src/lib/misc/timer.md#sleep)
* [waitTimeout](./src/lib/misc/timer.md#waitTimeout)
* [eachAsync](./src/lib/misc/each.md#eachAsync)
* [mapAsync](./src/lib/misc/map.md#mapAsync)
* [flatMapAsync](./src/lib/misc/map.md#flatMapAsync)
* [mapObjectAsync](./src/lib/misc/map.md#mapObjectAsync)
* [reduceAsync](./src/lib/misc/reduce.md#reduceAsync)
* [filterAsync](./src/lib/misc/filter.md#filterAsync)
* [someAsync](./src/lib/misc/some.md#someAsync)
* [everyAsync](./src/lib/misc/every.md#everyAsync)
* [parallelAsync](./src/lib/misc/parallel.md#parallelAsync)

## Related projects

* [f-streams-async](https://github.com/s.berthier/f-streams-async)

## License

MIT.
