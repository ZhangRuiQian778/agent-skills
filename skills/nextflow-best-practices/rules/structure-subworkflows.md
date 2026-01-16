---
title: Use Subworkflows for Reusable Stages
impact: MEDIUM-HIGH
impactDescription: Encapsulates multi-step logic
tags: structure, subworkflow, reuse
---

## Use Subworkflows for Reusable Stages

**Impact: MEDIUM-HIGH (Encapsulates multi-step logic)**

Group related steps into a subworkflow to reuse across pipelines and keep main workflows simple.

**Incorrect (repeat multiple steps in main workflow):**

```nextflow
workflow {
  reads_ch | TRIM | QC
  // ...
}
```

**Correct (define and reuse a subworkflow):**

```nextflow
workflow QC_AND_TRIM {
  take:
    reads_ch
  main:
    reads_ch | TRIM | QC
  emit:
    qc_ch
}

workflow {
  QC_AND_TRIM(reads_ch)
}
```

Reference: Nextflow subworkflow documentation
