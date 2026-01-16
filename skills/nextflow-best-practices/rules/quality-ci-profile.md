---
title: Provide a Minimal CI Profile
impact: MEDIUM
impactDescription: Makes CI fast and deterministic
tags: quality, ci, profiles
---

## Provide a Minimal CI Profile

**Impact: MEDIUM (Makes CI fast and deterministic)**

Define a CI profile with small resources and limited parallelism so tests run quickly and reliably.

**Incorrect (CI uses default heavy profile):**

```nextflow
// CI uses default profile with production resources
```

**Correct (dedicated CI profile):**

```nextflow
profiles {
  ci {
    process.cpus = 1
    process.memory = '1 GB'
    maxForks = 1
    params.small = true
  }
}
```

Reference: Nextflow configuration profiles documentation
