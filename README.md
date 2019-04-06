# Async iterable stream

This is a tiny library for creating async iterable objects:

```javascript
for await (const item of iter) {
  console.log('some async item')
}
```

from streams:

```javascript
const stream = require('awaitstream')()

// Send items into the stream
setInterval(() => stream.append(Date.now()), 1000)

// And consume them using a for-await loop:
for await (const time of stream.iter) {
  console.log('the time is', time)
}
```

Its similar to [streamiterator](https://www.npmjs.com/package/streamiterator), except:

- There is no out-of-the-box iteroperability with nodejs stream objects (because node streams are big and complicated)
- This library has no external dependancies. Minified and gzipped, this library adds only a couple hundred bytes to your bundle.


## Lifecycle

The stream has 2 states: Open and closed. The stream can be closed either from the producer or the consumer.

To close the stream from the producer, call `stream.end()`:

```javascript
const stream = streamToIter()
stream.append(1)
stream.append(2)
stream.end() // Producer indicates there are no more events to send into the stream

for await (const item of stream.iter) {
  console.log(item)
}
console.log('done')

// Prints 1, 2, done.
```

The consumer can indicate they are done consuming the stream by calling `iter.return()`:

When the consumer closes the stream, the stream will call a registered `onClosed` function. This function is registered by passing it as the first (and only) argument to the streamToIter constructor. (Eg `streamToIter(() => console.log('closed'))`)

*NOTE:* The onClosed function is only called if the consumer closes the stream via `iter.return()`. It is not called if the producer closes the stream via `stream.end()`.

```javascript
let timer
const stream = streamToIter(() => {
  clearInterval(timer) // Stop firing the interval once the consumer has finished reading from the stream
})
timer = setInterval(() => stream.append(Date.now()), 1000)

for await (const item of stream.iter) {
  if (Math.random() < 0.5) break
}
stream.iter.return()
```


## Handing errors

You can also inject errors into the stream. Errors will cause an exception to be thrown from the `for await` loop:

```javascript
const stream = streamToIter()
stream.throw(new Error('oops!'))

// ...


for await (const item of stream.iter) {

} // throws Error('oops!')!
```

Once an error is injected into the stream, the stream is considered to be done. No further events can be injected into the stream. The stream currently *does* call a registered onClose handler here - I'm not sure if thats the right call but this decision will not change without a bump to the major version of this library.


## Using streamtoiter without for-await support

Some older javascript implementations (node 8, IE11, etc) don't support for-await syntax. To use this library, either use a transpiler like babel or typescript, or iterate over the stream using one of these polyfill-esque syntaxes:

```javascript
// Option 1: adapt a for loop:
for (let item = await stream.iter.next(); !item.done; item = await stream.iter.next()) {
  console.log(item.value)
}

// Option 2: Adapt a while loop:
while(true) {
  let item = await stream.iter.next()
  if (item.done) break

  console.log(item.value)
}
```

Or some variant thereof.

