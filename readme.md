# Async/Await in LB Stanza

This repository contains a simple implementation of async/await on top of coroutines in stanza.

Usage:

```
; Asynchronous code can be called like normal synchronous code in async contexts, wrapped in
; calls to `await`. 
defn my-async () -> Awaitable:
  within await = async():
    ; ... 
    val this = await(that) as Type
    ; ... 

; Manually implementing an awaitable using promises. This API is primarily for
; implementing asynchronous libraries, and should not be necessary in application
; code
defn my-awaitable () -> Awaitable:
  within promise = awaitable():
    if condition?: 
      ; in case of failure, reject the promise with an exception.
      reject(promise, Exception("condition? was true"))
    else if other-condition?:
      ; promises are wrapped in try/catch blocks to convert exceptions into
      ; calls to `reject`. 
      throw(Exception("failed")
    else:
      ; You may suspend execution of the promise explicitly. 
      pend(promise)
      if final-condition?:
        ; You may explicitly resolve the promise
        resolve(promise, result) 
    ; the return of the function will implicitly resolve the promise as well
    other-result

defn main (): 
  ; awaitables must be driven using an executor. `execute` will evaulate them concurrently.
  execute([my-async(), my-awaitable()])
```

## Implementation Details: 

- Promises are implemented using stackful coroutines (`Coroutine<False, PromiseState>`). 
- Coroutines are constructed when converting an `Awaitable` to a `Promise` using `make-promise`.
- Within any instance of `Awaitable`, each subsequent call to `await` will suspend the top-level Coroutine. In other words, nested calls to `await` inside asynchronous contexts will all suspend the same coroutine (as opposed to each async context being assigned its own coroutine).
