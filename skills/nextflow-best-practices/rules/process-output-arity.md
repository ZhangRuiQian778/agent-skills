---
title: Enforce Output File Counts with arity
impact: MEDIUM-HIGH
impactDescription: Catches missing or extra outputs early
tags: process, output, arity, validation
---

## Enforce Output File Counts with arity

Use the `arity` option on `path` outputs to validate the expected number of files and fail fast when outputs are missing or excessive.

**Incorrect (no validation of output count):**

```nextflow
process SPLIT {
  output:
  path 'chunk_*.txt'

  script:
  """
  split -b 10 input.txt chunk_
  """
}
```

**Correct (assert expected arity):**

```nextflow
process SPLIT {
  output:
  path('chunk_*.txt', arity: '1..*')

  script:
  """
  split -b 10 input.txt chunk_
  """
}
```

Reference: Nextflow process output arity documentation
