---
title: Retry Transient Failures with errorStrategy
impact: HIGH
impactDescription: Reduces flaky failures on shared infrastructure
tags: process, errorStrategy, retries, resilience
---

## Retry Transient Failures with errorStrategy

Use `errorStrategy` with `maxRetries` to handle transient failures, and keep permanent failures fast-fail.

**Incorrect (always terminate on transient failure):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

**Correct (retry only on transient exit codes):**

```nextflow
process ALIGN {
  maxRetries 3
  errorStrategy { task.exitStatus in [137, 143] ? 'retry' : 'terminate' }

  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on error handling
