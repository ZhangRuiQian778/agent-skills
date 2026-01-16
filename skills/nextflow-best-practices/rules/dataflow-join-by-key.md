---
title: Join Channels by Key to Prevent Mispairing
impact: HIGH
impactDescription: Ensures correct pairing of related inputs
tags: dataflow, join, combine, keys
---

## Join Channels by Key to Prevent Mispairing

**Impact: HIGH (Ensures correct pairing of related inputs)**

Use join (or keyed combine) on a shared key to avoid relying on implicit ordering.

**Incorrect (order-dependent pairing):**

```nextflow
paired = reads.combine(bams)
```

**Correct (keyed join by sample_id):**

```nextflow
reads = Channel.fromPath(params.reads)
  .map { f -> tuple(f.baseName, f) }

bams = Channel.fromPath(params.bams)
  .map { f -> tuple(f.baseName, f) }

paired = reads.join(bams)

process MERGE {
  input:
  tuple val(sample_id), path(reads), path(bam)
  // ...
}
```

Reference: Nextflow join/combine documentation
