# Stream to iter

This is a tiny library for creating an async iterable object:

```javascript
for await (const item of iter) {
  console.log('some async item')
}
```

from a stream:

```javascript
const stream = require('streamtoiter')()

setInterval(() => stream.append(Date.now()), 1000)

for await (const time of stream.iter) {
  console.log('the time is', time)
}
```

