---
title: Document Inputs, Outputs, and Parameters
impact: MEDIUM
impactDescription: Improves usability and supportability
tags: quality, docs, params
---

## Document Inputs, Outputs, and Parameters

**Impact: MEDIUM (Improves usability and supportability)**

Clearly document required inputs, outputs, and parameters in README or parameter files so users run the pipeline correctly.

**Incorrect (params not documented):**

```nextflow
// No parameter documentation for users
```

**Correct (document parameters and defaults):**

```nextflow
// nextflow.config
params {
  reads = null   // input reads glob
  outdir = 'results'
}
```

Reference: Nextflow documentation guidelines
