---
title: Encourage -resume with Stable workDir
impact: MEDIUM-HIGH
impactDescription: Speeds reruns and saves compute
tags: cache, resume, workdir
---

## Encourage -resume with Stable workDir

**Impact: MEDIUM-HIGH (Speeds reruns and saves compute)**

Use a stable workDir and recommend -resume for reruns to reuse cached results.

**Incorrect (ephemeral workDir, no resume guidance):**

```nextflow
workDir = '/tmp/nextflow-work'
```

**Correct (stable workDir for caching):**

```nextflow
workDir = "/scratch/${System.getenv('USER')}/nextflow-work"
```

Reference: Nextflow resume and caching documentation
