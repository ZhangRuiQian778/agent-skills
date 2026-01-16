---
title: Enable Trace, Report, Timeline, and DAG
impact: MEDIUM-HIGH
impactDescription: Preserves provenance and diagnostics
tags: cache, provenance, trace, report
---

## Enable Trace, Report, Timeline, and DAG

**Impact: MEDIUM-HIGH (Preserves provenance and diagnostics)**

Enable built-in reports to capture run metadata and support troubleshooting.

**Incorrect (no provenance artifacts):**

```nextflow
// No trace/report/timeline settings
```

**Correct (enable in config or CLI):**

```nextflow
// nextflow.config
trace.enabled = true
trace.file = 'trace.txt'
report.enabled = true
report.file = 'report.html'
timeline.enabled = true
timeline.file = 'timeline.html'
dag.enabled = true
dag.file = 'dag.html'
```

Reference: Nextflow reporting documentation
