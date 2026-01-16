---
name: nextflow-best-practices
description: Nextflow pipeline best practices for DSL2 workflow structure, dataflow, process design, execution tuning, reproducibility, caching, and operational quality. Use when designing, reviewing, or refactoring Nextflow pipelines, configs, or process definitions.
---

# Nextflow Best Practices

Comprehensive best-practice guide for Nextflow DSL2 pipelines. Contains 32 rules across 8 categories, prioritized by impact to guide design, review, and refactoring.

## When to Apply

Reference these guidelines when:
- Designing or extending DSL2 workflows
- Reviewing pipeline reliability, performance, or cost
- Refactoring processes, modules, or subworkflows
- Tuning execution profiles and resource directives
- Hardening reproducibility, caching, and provenance

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Process Design & Resources | CRITICAL | `process-` |
| 2 | Execution & Scaling | HIGH | `exec-` |
| 3 | Dataflow & Metadata | HIGH | `dataflow-` |
| 4 | Environment & Reproducibility | HIGH | `env-` |
| 5 | Workflow Structure & Modularity | MEDIUM-HIGH | `structure-` |
| 6 | Configuration & Profiles | MEDIUM-HIGH | `config-` |
| 7 | Caching & Provenance | MEDIUM-HIGH | `cache-` |
| 8 | Quality & Operations | MEDIUM | `quality-` |

## Quick Reference

### 1. Process Design & Resources (CRITICAL)

- `process-declare-io` - Explicitly declare input/output types (val/path/tuple)
- `process-resource-directives` - Set cpus/memory/time per process
- `process-error-retry` - Use errorStrategy/maxRetries for transient failures
- `process-tag` - Add tag for traceability in logs and reports
- `process-avoid-loose-output-glob` - Avoid loose output globs like path '*'
- `process-output-arity` - Enforce output counts with arity to fail fast

### 2. Execution & Scaling (HIGH)

- `exec-profiles` - Use profiles per environment (local/slurm/cloud)
- `exec-maxforks` - Cap maxForks to avoid overload
- `exec-workdir-fast` - Place workDir on fast local or scratch storage

### 3. Dataflow & Metadata (HIGH)

- `dataflow-tuple-metadata` - Keep sample metadata with data as tuples
- `dataflow-join-by-key` - Join/combine channels using keys to avoid mispairing
- `dataflow-avoid-collect` - Avoid collect unless full materialization is required
- `dataflow-single-channel-inputs` - Combine channels before invoking multi-input processes

### 4. Environment & Reproducibility (HIGH)

- `env-container-pin` - Pin container versions or digests
- `env-per-process` - Define container/conda per process; avoid global mismatch
- `env-conda-lock` - Use explicit conda env files or lockfiles

### 5. Workflow Structure & Modularity (MEDIUM-HIGH)

- `structure-dsl2-entrypoint` - Keep a thin main workflow entry
- `structure-modules` - Extract processes into modules
- `structure-subworkflows` - Use subworkflows for reusable stages
- `structure-workflow-outputs` - Prefer workflow outputs over publishDir (25.10+)

### 6. Configuration & Profiles (MEDIUM-HIGH)

- `config-params-defaults` - Provide param defaults and validation
- `config-includeconfig` - Split config files by concern
- `config-secrets` - Keep secrets in env or secret store
- `config-label-selectors` - Use labels and withLabel for resource classes

### 7. Caching & Provenance (MEDIUM-HIGH)

- `cache-resume` - Encourage -resume with stable workDir
- `cache-deterministic` - Avoid nondeterministic outputs for cache hits
- `cache-avoid-modifying-inputs` - Never modify input files in-place
- `cache-avoid-global-state` - Avoid global mutable state in operators
- `cache-trace-report` - Enable trace/report/timeline for provenance

### 8. Quality & Operations (MEDIUM)

- `quality-tests` - Add automated tests with small fixtures
- `quality-ci-profile` - Provide a minimal CI profile
- `quality-docs-params` - Document inputs/outputs and parameter meaning

## How to Use

Read individual rule files for detailed explanations and examples:

```
rules/process-resource-directives.md
rules/dataflow-join-by-key.md
rules/_sections.md
```

Each rule file contains:
- Brief rationale
- Incorrect example
- Correct example
- Notes and references
