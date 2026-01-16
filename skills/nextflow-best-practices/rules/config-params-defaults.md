---
title: Provide Parameter Defaults and Validation
impact: MEDIUM-HIGH
impactDescription: Prevents silent misconfigurations
tags: config, params, validation
---

## Provide Parameter Defaults and Validation

**Impact: MEDIUM-HIGH (Prevents silent misconfigurations)**

Define reasonable defaults and validate required params early to fail fast.

**Incorrect (no defaults or validation):**

```nextflow
workflow {
  reads_ch = Channel.fromPath(params.reads)
}
```

**Correct (defaults and validation):**

```nextflow
params.outdir = params.outdir ?: 'results'

if (!params.reads)
  error "Missing required --reads"

workflow {
  reads_ch = Channel.fromPath(params.reads)
}
```

Reference: Nextflow parameters documentation
