---
title: Add Automated Pipeline Tests with Small Fixtures
impact: MEDIUM
impactDescription: Catches regressions early
tags: quality, tests, ci
---

## Add Automated Pipeline Tests with Small Fixtures

**Impact: MEDIUM (Catches regressions early)**

Include minimal test data and run the pipeline in a lightweight test profile in CI.

**Incorrect (manual-only testing):**

```nextflow
// No test inputs or automated runs
```

**Correct (small fixtures and test profile):**

```nextflow
// nextflow.config
profiles {
  test {
    params.reads = 'tests/data/*.fastq.gz'
    process.cpus = 1
    process.memory = '2 GB'
    maxForks = 1
  }
}
```

Reference: Nextflow testing best practices
