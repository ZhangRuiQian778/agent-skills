---
title: Avoid Loose Output Globs
impact: HIGH
impactDescription: Prevents unnecessary file copies and unexpected outputs
tags: process, output, glob, io
---

## Avoid Loose Output Globs

Avoid broad patterns like `path '*'` because they can trigger unnecessary file copies from the scratch directory and emit unintended files. Use specific prefixes or suffixes instead.

**Incorrect (too broad):**

```nextflow
process SORT {
  output:
  path '*'

  script:
  """
  sort input.txt > prefix_sorted.txt
  """
}
```

**Correct (restrict to expected outputs):**

```nextflow
process SORT {
  output:
  path 'prefix_*.txt'

  script:
  """
  sort input.txt > prefix_sorted.txt
  """
}
```

Reference: Nextflow process output glob documentation
