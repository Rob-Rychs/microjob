# API
This documentation helps using `microjob` with ease, involving simple and quick examples.

## Sync job
The common and most used example is the sync job.
A sync job is just a function working in background, in another thread, avoiding to block the main thread with heavy CPU load, made of sync function calls.

```js
(async () => {
  const { job } = require('microjob')

  try {
    // this function will be executed in another thread
    const res = await job(() => {
      let i = 0
      for (i = 0; i < 1000000; i++) {
        for (let j = 0; j < 1000000; j++) {
          for (let k = 0; k < 1000000; k++) {}
        }
      }

      return i
    })

    console.log(res) // 1000000
  } catch (err) {
    console.error(err)
  }
})()
```

## Async job
An asynchronous job is a task with at least one async call: for instance, a query to a DB, a HTTP request, a file system call, etc, and of course plus a heavy CPU load.

```js
(async () => {
  const { job } = require('microjob')

  try {
    // this function will be executed in another thread
    const res = await job(async () => {
      let i = 0
      for (i = 0; i < 1000000; i++) {
        for (let j = 0; j < 1000000; j++) {
          for (let k = 0; k < 1000000; k++) {
            await http.get('www.google.it')
          }
        }
      }

      return i
    })

    console.log(res) // 1000000
  } catch (err) {
    console.error(err)
  }
})()
```

## Job data
Passing custom data to the job is quite easy as calling a function:

```js
(async () => {
  const { job } = require('microjob')

  try {
    // this function will be executed in another thread
    const res = await job(data => {
      let i = 0
      for (i = 0; i < data.counter; i++) {}

      return i
    }, {data: {counter: 1000000}})

    console.log(res) // 1000000
  } catch (err) {
    console.error(err)
  }
})()
```

Both data passed to the worker and the returned one are serialized with [v8 serializer](https://nodejs.org/api/v8.html#v8_v8_serialize_value): this means that only some JS data structures are allowed (for instance, functions and classes are forbidden).

## Job context
It's a common practice using the upper scope of the function's container to reuse the already defined variables.

**Context is evaluated inside the worker thread. This means it needs to be sanitized before passing it to microjob.
An attacker could perform a JS injection as described [in this issue](https://github.com/wilk/microjob/issues/2)**

Achieving the same result can be done by passing the context object:

```js
(async () => {
  const { job } = require('microjob')

  try {
    // this function will be executed in another thread
    const counter = 1000000
    const res = await job(() => {
      let i = 0
      for (i = 0; i < counter; i++) {}

      return i
    }, {ctx: {counter}})

    console.log(res) // 1000000
  } catch (err) {
    console.error(err)
  }
})()
```
