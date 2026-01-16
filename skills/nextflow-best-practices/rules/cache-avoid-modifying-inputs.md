---
title: Never Modify Input Files In-Place
impact: HIGH
impactDescription: Preserves resumability and cache validity
tags: cache, inputs, reproducibility
---

## Never Modify Input Files In-Place

Processes that modify their own input files cannot be resumed. Treat inputs as read-only and write outputs to new files.

**Incorrect (mutates input file):**

```nextflow
process CLEAN {
  input:
  path(reads)

  output:
  path(reads)

  script:
  """
  tr -d 'N' < $reads > $reads
  """
}
```

**Correct (write to a new output):**

```nextflow
process CLEAN {
  input:
  path(reads)

  output:
  path('cleaned.fastq')

  script:
  """
  tr -d 'N' < $reads > cleaned.fastq
  """
}
```

Reference: Nextflow cache and resume documentation
