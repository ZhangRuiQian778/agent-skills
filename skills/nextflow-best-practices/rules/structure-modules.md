---
title: Extract Processes into Modules
impact: MEDIUM-HIGH
impactDescription: Enables reuse and testing
tags: structure, modules, process
---

## Extract Processes into Modules

**Impact: MEDIUM-HIGH (Enables reuse and testing)**

Move each process into a module file and include it where needed.

**Incorrect (process defined inline):**

```nextflow
process ALIGN {
  // ...
}
```

**Correct (module file and include):**

```nextflow
// modules/align.nf
process ALIGN {
  // ...
}

// main.nf
include { ALIGN } from './modules/align'
```

Reference: Nextflow modules documentation
