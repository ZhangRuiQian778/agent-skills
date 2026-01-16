---
title: Set Resource Directives Per Process
impact: CRITICAL
impactDescription: Improves scheduling, prevents oversubscription
tags: process, resources, cpus, memory, time
---

## Set Resource Directives Per Process

Declare `cpus`, `memory`, and `time` for each process and align tool flags with `task.*` to avoid under/over-allocation.

**Incorrect (tool uses many threads but no resource declaration):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner -t 16 $reads > aligned.bam
  """
}
```

**Correct (declare resources and bind to task.cpus):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  time '2h'

  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on process directives
