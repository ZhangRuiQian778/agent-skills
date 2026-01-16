---
title: Explicitly Declare Input/Output Types
impact: CRITICAL
impactDescription: Prevents sample mix-ups and implicit staging
tags: process, io, tuple, metadata
---

## Explicitly Declare Input/Output Types

Declare `val`, `path`, and `tuple` explicitly so data and metadata stay aligned and outputs are unambiguous.

**Incorrect (loses sample metadata and produces ambiguous outputs):**

```nextflow
process ALIGN {
  input:
  path reads

  output:
  path "aligned.bam"

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

**Correct (keep metadata with files using tuple):**

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

Reference: Nextflow documentation on process inputs/outputs
