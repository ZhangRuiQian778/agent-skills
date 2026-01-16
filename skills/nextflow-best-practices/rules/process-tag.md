---
title: Add Tags for Traceability
impact: MEDIUM
impactDescription: Improves log and report readability
tags: process, tag, observability
---

## Add Tags for Traceability

Use `tag` to include sample or task identifiers in logs, trace, and timeline outputs.

**Incorrect (logs are hard to correlate):**

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

**Correct (tag includes sample id):**

```nextflow
process ALIGN {
  tag "${sample_id}"

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

Reference: Nextflow documentation on process tags
