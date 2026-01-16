---
title: Prefer Workflow Outputs for Publishing (25.10+)
impact: MEDIUM-HIGH
impactDescription: Makes outputs explicit and reduces publishDir misuse
tags: structure, workflow, outputs, publish
---

## Prefer Workflow Outputs for Publishing (25.10+)

Workflow outputs are intended to replace `publishDir` in modern DSL2 pipelines. Declare outputs in the workflow and publish them explicitly. For older versions, use `publishDir` with `saveAs` to avoid collisions.

**Incorrect (only publishDir, outputs implicit):**

```nextflow
process FETCH {
  publishDir 'results'

  output:
  path 'sample.txt'
}
```

**Correct (workflow outputs + publish section):**

```nextflow
process FETCH {
  output:
  path 'sample.txt'
}

workflow {
  main:
  ch_samples = FETCH(params.input)

  publish:
  samples = ch_samples
}

output {
  samples {
    path '.'
  }
}
```

Reference: Nextflow workflow outputs documentation
