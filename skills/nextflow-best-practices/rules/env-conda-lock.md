---
title: Use Explicit Conda Environments or Lockfiles
impact: HIGH
impactDescription: Avoids dependency drift across runs
tags: env, conda, reproducibility
---

## Use Explicit Conda Environments or Lockfiles

**Impact: HIGH (Avoids dependency drift across runs)**

Prefer explicit environment files or pinned package versions to keep conda environments reproducible.

**Incorrect (unversioned package spec):**

```nextflow
process ALIGN {
  conda 'bioconda::bwa'
  // ...
}
```

**Correct (use env file or pin versions):**

```nextflow
process ALIGN {
  conda 'envs/bwa.yml'
  // ...
}
```

Reference: Nextflow conda documentation
