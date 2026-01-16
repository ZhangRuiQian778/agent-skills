---
title: Place workDir on Fast Local or Scratch Storage
impact: HIGH
impactDescription: Avoids slow shared filesystem bottlenecks
tags: exec, workdir, io, storage
---

## Place workDir on Fast Local or Scratch Storage

**Impact: HIGH (Avoids slow shared filesystem bottlenecks)**

Use a fast local or scratch path for workDir to improve task staging and reduce NFS contention.

**Incorrect (workDir on shared network storage):**

```nextflow
// nextflow.config
workDir = '/mnt/shared/nextflow-work'
```

**Correct (workDir on local or scratch storage):**

```nextflow
// nextflow.config
workDir = "/scratch/${System.getenv('USER')}/nextflow-work"
```

Reference: Nextflow work directory documentation
