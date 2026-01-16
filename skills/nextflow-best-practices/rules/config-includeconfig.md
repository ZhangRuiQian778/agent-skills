---
title: Split Configuration with includeConfig
impact: MEDIUM-HIGH
impactDescription: Keeps config files maintainable
tags: config, includeConfig, profiles
---

## Split Configuration with includeConfig

**Impact: MEDIUM-HIGH (Keeps config files maintainable)**

Use includeConfig to split base settings, resources, and profiles into dedicated files.

**Incorrect (single monolithic config):**

```nextflow
// nextflow.config
// hundreds of lines here
process.executor = 'slurm'
// ...
```

**Correct (compose configs):**

```nextflow
// nextflow.config
includeConfig 'conf/base.config'
includeConfig 'conf/resources.config'
includeConfig 'conf/profiles.config'
```

Reference: Nextflow configuration include documentation
