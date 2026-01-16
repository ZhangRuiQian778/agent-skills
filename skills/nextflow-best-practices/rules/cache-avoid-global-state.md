---
title: Avoid Global Mutable State in Operators
impact: MEDIUM-HIGH
impactDescription: Prevents race conditions and nondeterministic cache keys
tags: cache, dataflow, globals, nondeterministic
---

## Avoid Global Mutable State in Operators

Using implicit global variables inside channel operators can create race conditions and non-deterministic values, which breaks caching. Always declare local variables with `def` inside closures.

**Incorrect (global mutable variable):**

```nextflow
Channel.of(1,2,3).map { v -> X=v; X+=2 }
Channel.of(1,2,3).map { v -> X=v; X*=2 }
```

**Correct (local variable per closure):**

```nextflow
Channel.of(1,2,3).map { v -> def x=v; x+=2 }
Channel.of(1,2,3).map { v -> def x=v; x*=2 }
```

Reference: Nextflow cache and resume documentation
