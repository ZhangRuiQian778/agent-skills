---
title: Combine Multi-Input Channels Before Invoking a Process
impact: HIGH
impactDescription: Avoids non-deterministic input pairing
tags: dataflow, channels, combine, nondeterministic
---

## Combine Multi-Input Channels Before Invoking a Process

Avoid calling a process with multiple channels directly, as it can produce non-deterministic input pairing. Combine or join channels first and pass a single channel of tuples.

**Incorrect (two channels passed directly):**

```nextflow
process MERGE {
  input:
  tuple val(sample_id), path(reads), path(bam)

  script:
  """
  merge_tool $reads $bam > merged.bam
  """
}

workflow {
  reads_ch = Channel.fromPath('reads/*.fq.gz')
    .map { f -> tuple(f.baseName, f) }
  bams_ch = Channel.fromPath('bams/*.bam')
    .map { f -> tuple(f.baseName, f) }

  MERGE(reads_ch, bams_ch)
}
```

**Correct (combine or join first):**

```nextflow
workflow {
  reads_ch = Channel.fromPath('reads/*.fq.gz')
    .map { f -> tuple(f.baseName, f) }
  bams_ch = Channel.fromPath('bams/*.bam')
    .map { f -> tuple(f.baseName, f) }

  merged_inputs = reads_ch.join(bams_ch)
  MERGE(merged_inputs)
}
```

Reference: Nextflow process multiple inputs warning
