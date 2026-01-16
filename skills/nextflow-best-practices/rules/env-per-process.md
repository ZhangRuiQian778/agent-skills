---
title: Define Container or Conda Per Process
impact: HIGH
impactDescription: Avoids tool version mismatches
tags: env, container, conda, process
---

## Define Container or Conda Per Process

**Impact: HIGH (Avoids tool version mismatches)**

Set container or conda per process so each tool runs in the expected environment.

**Incorrect (single global container for all tools):**

```nextflow
// nextflow.config
process.container = 'quay.io/biocontainers/toolbox:latest'
```

**Correct (per-process containers):**

```nextflow
process ALIGN {
  container 'quay.io/biocontainers/bwa:0.7.17--hed695b0_7'
  // ...
}

process SORT {
  container 'quay.io/biocontainers/samtools:1.17--h00cdaf9_0'
  // ...
}
```

Reference: Nextflow process container documentation
