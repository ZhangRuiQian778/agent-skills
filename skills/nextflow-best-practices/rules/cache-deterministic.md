---
title: Keep Outputs Deterministic for Cache Hits
impact: MEDIUM-HIGH
impactDescription: Avoids cache misses and drift
tags: cache, deterministic, reproducibility
---

## Keep Outputs Deterministic for Cache Hits

**Impact: MEDIUM-HIGH (Avoids cache misses and drift)**

Avoid nondeterministic outputs (timestamps, random names) so cache signatures remain stable.

**Incorrect (nondeterministic output content):**

```nextflow
process REPORT {
  output:
  path('report.txt')

  script:
  """
  generate_report --timestamp $(date +%s) > report.txt
  """
}
```

**Correct (deterministic output):**

```nextflow
process REPORT {
  output:
  path('report.txt')

  script:
  """
  generate_report --seed 42 > report.txt
  """
}
```

Reference: Nextflow caching documentation
