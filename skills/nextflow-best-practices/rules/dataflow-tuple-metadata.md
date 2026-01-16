---
title: Keep Sample Metadata with Data as Tuples
impact: HIGH
impactDescription: Preserves sample identity across processes
tags: dataflow, channels, tuples, metadata
---

## Keep Sample Metadata with Data as Tuples

**Impact: HIGH (Preserves sample identity across processes)**

Pair sample identifiers with their files in tuples so downstream steps can track and label outputs correctly.

**Incorrect (metadata lost when passing only files):**

```nextflow
Channel.fromPath(params.reads).set { reads }

process QC {
  input:
  path(reads)

  output:
  path("*.qc")
}
```

**Correct (keep sample_id and reads together):**

```nextflow
Channel.fromPath(params.reads)
  .map { file -> tuple(file.baseName, file) }
  .set { reads }

process QC {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("*.qc")
}
```

Reference: Nextflow channels and tuples documentation
