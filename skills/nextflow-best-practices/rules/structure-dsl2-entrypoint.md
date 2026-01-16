---
title: Keep a Thin DSL2 Entry Workflow
impact: MEDIUM-HIGH
impactDescription: Improves readability and reuse
tags: structure, dsl2, workflow
---

## Keep a Thin DSL2 Entry Workflow

**Impact: MEDIUM-HIGH (Improves readability and reuse)**

Keep main.nf as a lightweight entrypoint that wires modules and subworkflows instead of containing all logic.

**Incorrect (heavy logic in main.nf):**

```nextflow
process ALIGN {
  // ...
}

process SORT {
  // ...
}

workflow {
  reads_ch | ALIGN | SORT
}
```

**Correct (wire modules and subworkflows):**

```nextflow
include { ALIGN } from './modules/align'
include { SORT } from './modules/sort'

workflow {
  reads_ch | ALIGN | SORT
}
```

Reference: Nextflow DSL2 modules documentation
