---
title: Cap maxForks to Protect Shared Resources
impact: HIGH
impactDescription: Prevents scheduler overload and I/O storms
tags: exec, maxForks, parallelism, scheduler
---

## Cap maxForks to Protect Shared Resources

**Impact: HIGH (Prevents scheduler overload and I/O storms)**

Limit concurrent tasks for heavy processes with maxForks or withName selectors to avoid saturating clusters or shared filesystems.

**Incorrect (no cap on a heavy process):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'

  input:
  tuple val(sample_id), path(reads)

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

**Correct (cap concurrency for the heavy process):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  maxForks 4

  input:
  tuple val(sample_id), path(reads)

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

Reference: Nextflow process directives documentation
