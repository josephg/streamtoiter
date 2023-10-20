# 2.0.2

Tiny point release removing some lingering export code.

# 2.0.0

There aren't any semantic changes in version 2, since the library works great. But we've been running into problems with the typescript typing for the module, so I've:

- Moved the module to use ESM
- Removed the default export.

You'll now need to:

```typescript
import { createIterableStream } from 'ministreamiterator'
```

Version bumped to 2.0.0 for semver.