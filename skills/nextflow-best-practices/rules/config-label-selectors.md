---
title: Use Labels and withLabel for Resource Classes
impact: MEDIUM-HIGH
impactDescription: Centralizes resource policy and reduces duplication
tags: config, labels, selectors, resources
---

## Use Labels and withLabel for Resource Classes

Assign resource classes with process `label` and configure resources centrally using `withLabel` selectors. Use `withName` only for exceptions, and remember `withName` overrides `withLabel`.

**Incorrect (resources scattered in process bodies):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  ...
}
```

**Correct (labels + centralized selectors):**

```nextflow
process ALIGN {
  label 'big_mem'
  ...
}

// nextflow.config
process {
  withLabel: big_mem {
    cpus = 16
    memory = 32.GB
    queue = 'long'
  }
}
```

Reference: Nextflow process selectors documentation
