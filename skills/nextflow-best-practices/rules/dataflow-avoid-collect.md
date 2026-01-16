---
title: Avoid collect for Large Channels
impact: HIGH
impactDescription: Prevents memory blowups and preserves streaming
tags: dataflow, collect, streaming
---

## Avoid collect for Large Channels

**Impact: HIGH (Prevents memory blowups and preserves streaming)**

collect materializes the entire channel in memory and blocks streaming. Use streaming operators or bounded aggregations only when required.

**Incorrect (materializes all items before running):**

```nextflow
all_reads = Channel.fromPath(params.reads).collect()

process SUMMARIZE {
  input:
  path(all_reads)
  // ...
}
```

**Correct (stream per item instead of collecting):**

```nextflow
reads = Channel.fromPath(params.reads)

process QC {
  input:
  path(reads)
  // ...
}
```

Reference: Nextflow channel operators documentation
