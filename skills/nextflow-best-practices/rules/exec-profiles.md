---
title: Use Profiles Per Execution Environment
impact: HIGH
impactDescription: Prevents misconfigured executors and defaults
tags: exec, profiles, config, executor
---

## Use Profiles Per Execution Environment

**Impact: HIGH (Prevents misconfigured executors and defaults)**

Define per-environment profiles (local, slurm, cloud) so executor settings and resource defaults match the target system.

**Incorrect (single config used everywhere):**

```nextflow
// nextflow.config
process.executor = 'local'
process.cpus = 16
process.memory = '64 GB'
```

**Correct (explicit profiles and select with -profile):**

```nextflow
// nextflow.config
profiles {
  local {
    process.executor = 'local'
    process.cpus = 4
    process.memory = '8 GB'
  }

  slurm {
    process.executor = 'slurm'
    process.queue = 'batch'
    process.cpus = 32
    process.memory = '128 GB'
  }
}
```

Reference: Nextflow configuration profiles documentation
