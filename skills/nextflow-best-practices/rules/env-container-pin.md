---
title: Pin Container Versions or Digests
impact: HIGH
impactDescription: Ensures reproducible environments
tags: env, container, reproducibility
---

## Pin Container Versions or Digests

**Impact: HIGH (Ensures reproducible environments)**

Avoid floating tags like latest. Use explicit version tags or immutable digests.

**Incorrect (floating tag):**

```nextflow
process FASTQC {
  container 'quay.io/biocontainers/fastqc:latest'
  // ...
}
```

**Correct (pinned version or digest):**

```nextflow
process FASTQC {
  container 'quay.io/biocontainers/fastqc:0.11.9--0'
  // ...
}
```

Reference: Nextflow container documentation
