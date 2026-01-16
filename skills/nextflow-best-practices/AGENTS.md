# Nextflow Best Practices

**Version 0.1.0**  
Nextflow Project  
January 2026

> **Note:**  
> This document is mainly for agents and LLMs to follow when maintaining,  
> generating, or refactoring Nextflow DSL2 pipelines. Humans may also find  
> it useful, but guidance here is optimized for automation and consistency.

---

## Abstract

Comprehensive best-practice guide for Nextflow DSL2 pipelines, designed for AI agents and LLMs. Contains 32 rules across 8 categories, prioritized by impact from critical reliability and performance fixes to operational quality improvements. Each rule includes explanations, incorrect vs. correct examples, and references to guide automated refactoring and code generation.

---

## Table of Contents

1. [Process Design & Resources](#1-process-design-resources) — **CRITICAL**
   - 1.1 [Explicitly Declare Input/Output Types](#11-explicitly-declare-inputoutput-types)
   - 1.2 [Set Resource Directives Per Process](#12-set-resource-directives-per-process)
   - 1.3 [Retry Transient Failures with errorStrategy](#13-retry-transient-failures-with-errorstrategy)
   - 1.4 [Add Tags for Traceability](#14-add-tags-for-traceability)
   - 1.5 [Avoid Loose Output Globs](#15-avoid-loose-output-globs)
   - 1.6 [Enforce Output File Counts with arity](#16-enforce-output-file-counts-with-arity)
2. [Execution & Scaling](#2-execution-scaling) — **HIGH**
   - 2.1 [Use Profiles Per Execution Environment](#21-use-profiles-per-execution-environment)
   - 2.2 [Cap maxForks to Protect Shared Resources](#22-cap-maxforks-to-protect-shared-resources)
   - 2.3 [Place workDir on Fast Local or Scratch Storage](#23-place-workdir-on-fast-local-or-scratch-storage)
3. [Dataflow & Metadata](#3-dataflow-metadata) — **HIGH**
   - 3.1 [Keep Sample Metadata with Data as Tuples](#31-keep-sample-metadata-with-data-as-tuples)
   - 3.2 [Join Channels by Key to Prevent Mispairing](#32-join-channels-by-key-to-prevent-mispairing)
   - 3.3 [Avoid collect for Large Channels](#33-avoid-collect-for-large-channels)
   - 3.4 [Combine Multi-Input Channels Before Invoking a Process](#34-combine-multi-input-channels-before-invoking-a-process)
4. [Environment & Reproducibility](#4-environment-reproducibility) — **HIGH**
   - 4.1 [Pin Container Versions or Digests](#41-pin-container-versions-or-digests)
   - 4.2 [Define Container or Conda Per Process](#42-define-container-or-conda-per-process)
   - 4.3 [Use Explicit Conda Environments or Lockfiles](#43-use-explicit-conda-environments-or-lockfiles)
5. [Workflow Structure & Modularity](#5-workflow-structure-modularity) — **MEDIUM-HIGH**
   - 5.1 [Keep a Thin DSL2 Entry Workflow](#51-keep-a-thin-dsl2-entry-workflow)
   - 5.2 [Extract Processes into Modules](#52-extract-processes-into-modules)
   - 5.3 [Use Subworkflows for Reusable Stages](#53-use-subworkflows-for-reusable-stages)
   - 5.4 [Prefer Workflow Outputs for Publishing (25.10+)](#54-prefer-workflow-outputs-for-publishing-2510)
6. [Configuration & Profiles](#6-configuration-profiles) — **MEDIUM-HIGH**
   - 6.1 [Provide Parameter Defaults and Validation](#61-provide-parameter-defaults-and-validation)
   - 6.2 [Split Configuration with includeConfig](#62-split-configuration-with-includeconfig)
   - 6.3 [Keep Secrets Out of Versioned Config](#63-keep-secrets-out-of-versioned-config)
   - 6.4 [Use Labels and withLabel for Resource Classes](#64-use-labels-and-withlabel-for-resource-classes)
7. [Caching & Provenance](#7-caching-provenance) — **MEDIUM-HIGH**
   - 7.1 [Encourage -resume with Stable workDir](#71-encourage-resume-with-stable-workdir)
   - 7.2 [Keep Outputs Deterministic for Cache Hits](#72-keep-outputs-deterministic-for-cache-hits)
   - 7.3 [Never Modify Input Files In-Place](#73-never-modify-input-files-in-place)
   - 7.4 [Avoid Global Mutable State in Operators](#74-avoid-global-mutable-state-in-operators)
   - 7.5 [Enable Trace, Report, Timeline, and DAG](#75-enable-trace-report-timeline-and-dag)
8. [Quality & Operations](#8-quality-operations) — **MEDIUM**
   - 8.1 [Add Automated Pipeline Tests with Small Fixtures](#81-add-automated-pipeline-tests-with-small-fixtures)
   - 8.2 [Provide a Minimal CI Profile](#82-provide-a-minimal-ci-profile)
   - 8.3 [Document Inputs, Outputs, and Parameters](#83-document-inputs-outputs-and-parameters)

---

## 1. Process Design & Resources

**Impact: CRITICAL**

Correct inputs/outputs and right-sized resources are the largest drivers of pipeline reliability and performance.

### 1.1 Explicitly Declare Input/Output Types

**Impact: CRITICAL (Prevents sample mix-ups and implicit staging)**

Declare `val`, `path`, and `tuple` explicitly so data and metadata stay aligned and outputs are unambiguous.

**Incorrect (loses sample metadata and produces ambiguous outputs):**

```nextflow
process ALIGN {
  input:
  path reads

  output:
  path "aligned.bam"

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

**Correct (keep metadata with files using tuple):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on process inputs/outputs

### 1.2 Set Resource Directives Per Process

**Impact: CRITICAL (Improves scheduling, prevents oversubscription)**

Declare `cpus`, `memory`, and `time` for each process and align tool flags with `task.*` to avoid under/over-allocation.

**Incorrect (tool uses many threads but no resource declaration):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner -t 16 $reads > aligned.bam
  """
}
```

**Correct (declare resources and bind to task.cpus):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  time '2h'

  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on process directives

### 1.3 Retry Transient Failures with errorStrategy

**Impact: HIGH (Reduces flaky failures on shared infrastructure)**

Use `errorStrategy` with `maxRetries` to handle transient failures, and keep permanent failures fast-fail.

**Incorrect (always terminate on transient failure):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

**Correct (retry only on transient exit codes):**

```nextflow
process ALIGN {
  maxRetries 3
  errorStrategy { task.exitStatus in [137, 143] ? 'retry' : 'terminate' }

  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on error handling

### 1.4 Add Tags for Traceability

**Impact: MEDIUM (Improves log and report readability)**

Use `tag` to include sample or task identifiers in logs, trace, and timeline outputs.

**Incorrect (logs are hard to correlate):**

```nextflow
process ALIGN {
  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

**Correct (tag includes sample id):**

```nextflow
process ALIGN {
  tag "${sample_id}"

  input:
  tuple val(sample_id), path(reads)

  output:
  tuple val(sample_id), path("aligned.bam")

  script:
  """
  aligner $reads > aligned.bam
  """
}
```

Reference: Nextflow documentation on process tags

### 1.5 Avoid Loose Output Globs

**Impact: HIGH (Prevents unnecessary file copies and unexpected outputs)**

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

### 1.6 Enforce Output File Counts with arity

**Impact: MEDIUM-HIGH (Catches missing or extra outputs early)**

Use the `arity` option on `path` outputs to validate the expected number of files and fail fast when outputs are missing or excessive.

**Incorrect (no validation of output count):**

```nextflow
process SPLIT {
  output:
  path 'chunk_*.txt'

  script:
  """
  split -b 10 input.txt chunk_
  """
}
```

**Correct (assert expected arity):**

```nextflow
process SPLIT {
  output:
  path('chunk_*.txt', arity: '1..*')

  script:
  """
  split -b 10 input.txt chunk_
  """
}
```

Reference: Nextflow process output arity documentation

## 2. Execution & Scaling

**Impact: HIGH**

Executor and parallelism tuning prevent overload and shorten time-to-results.

### 2.1 Use Profiles Per Execution Environment

**Impact: HIGH (Prevents misconfigured executors and defaults)**

Define per-environment profiles (local, slurm, cloud) so executor settings and resource defaults match the target system.

**Incorrect (single config used everywhere):**

```nextflow
// nextflow.config
process.executor = 'local'
process.cpus = 16
process.memory = '64 GB'
```

**Correct (explicit profiles and select with -profile):**

```nextflow
// nextflow.config
profiles {
  local {
    process.executor = 'local'
    process.cpus = 4
    process.memory = '8 GB'
  }

  slurm {
    process.executor = 'slurm'
    process.queue = 'batch'
    process.cpus = 32
    process.memory = '128 GB'
  }
}
```

Reference: Nextflow configuration profiles documentation

### 2.2 Cap maxForks to Protect Shared Resources

**Impact: HIGH (Prevents scheduler overload and I/O storms)**

Limit concurrent tasks for heavy processes with maxForks or withName selectors to avoid saturating clusters or shared filesystems.

**Incorrect (no cap on a heavy process):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'

  input:
  tuple val(sample_id), path(reads)

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

**Correct (cap concurrency for the heavy process):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  maxForks 4

  input:
  tuple val(sample_id), path(reads)

  script:
  """
  aligner -t ${task.cpus} $reads > aligned.bam
  """
}
```

Reference: Nextflow process directives documentation

### 2.3 Place workDir on Fast Local or Scratch Storage

**Impact: HIGH (Avoids slow shared filesystem bottlenecks)**

Use a fast local or scratch path for workDir to improve task staging and reduce NFS contention.

**Incorrect (workDir on shared network storage):**

```nextflow
// nextflow.config
workDir = '/mnt/shared/nextflow-work'
```

**Correct (workDir on local or scratch storage):**

```nextflow
// nextflow.config
workDir = "/scratch/${System.getenv('USER')}/nextflow-work"
```

Reference: Nextflow work directory documentation

## 3. Dataflow & Metadata

**Impact: HIGH**

Stable channel structure and keyed joins reduce data mismatches and enable scalable streaming.

### 3.1 Keep Sample Metadata with Data as Tuples

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

### 3.2 Join Channels by Key to Prevent Mispairing

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

### 3.3 Avoid collect for Large Channels

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

### 3.4 Combine Multi-Input Channels Before Invoking a Process

**Impact: HIGH (Avoids non-deterministic input pairing)**

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

## 4. Environment & Reproducibility

**Impact: HIGH**

Pinning runtime environments ensures reproducible results across machines and time.

### 4.1 Pin Container Versions or Digests

**Impact: HIGH (Ensures reproducible environments)**

Avoid floating tags like latest. Use explicit version tags or immutable digests.

**Incorrect (floating tag):**

```nextflow
process FASTQC {
  container 'quay.io/biocontainers/fastqc:latest'
  // ...
}
```

**Correct (pinned version or digest):**

```nextflow
process FASTQC {
  container 'quay.io/biocontainers/fastqc:0.11.9--0'
  // ...
}
```

Reference: Nextflow container documentation

### 4.2 Define Container or Conda Per Process

**Impact: HIGH (Avoids tool version mismatches)**

Set container or conda per process so each tool runs in the expected environment.

**Incorrect (single global container for all tools):**

```nextflow
// nextflow.config
process.container = 'quay.io/biocontainers/toolbox:latest'
```

**Correct (per-process containers):**

```nextflow
process ALIGN {
  container 'quay.io/biocontainers/bwa:0.7.17--hed695b0_7'
  // ...
}

process SORT {
  container 'quay.io/biocontainers/samtools:1.17--h00cdaf9_0'
  // ...
}
```

Reference: Nextflow process container documentation

### 4.3 Use Explicit Conda Environments or Lockfiles

**Impact: HIGH (Avoids dependency drift across runs)**

Prefer explicit environment files or pinned package versions to keep conda environments reproducible.

**Incorrect (unversioned package spec):**

```nextflow
process ALIGN {
  conda 'bioconda::bwa'
  // ...
}
```

**Correct (use env file or pin versions):**

```nextflow
process ALIGN {
  conda 'envs/bwa.yml'
  // ...
}
```

Reference: Nextflow conda documentation

## 5. Workflow Structure & Modularity

**Impact: MEDIUM-HIGH**

Modular workflows are easier to test, reuse, and maintain as pipelines evolve.

### 5.1 Keep a Thin DSL2 Entry Workflow

**Impact: MEDIUM-HIGH (Improves readability and reuse)**

Keep main.nf as a lightweight entrypoint that wires modules and subworkflows instead of containing all logic.

**Incorrect (heavy logic in main.nf):**

```nextflow
process ALIGN {
  // ...
}

process SORT {
  // ...
}

workflow {
  reads_ch | ALIGN | SORT
}
```

**Correct (wire modules and subworkflows):**

```nextflow
include { ALIGN } from './modules/align'
include { SORT } from './modules/sort'

workflow {
  reads_ch | ALIGN | SORT
}
```

Reference: Nextflow DSL2 modules documentation

### 5.2 Extract Processes into Modules

**Impact: MEDIUM-HIGH (Enables reuse and testing)**

Move each process into a module file and include it where needed.

**Incorrect (process defined inline):**

```nextflow
process ALIGN {
  // ...
}
```

**Correct (module file and include):**

```nextflow
// modules/align.nf
process ALIGN {
  // ...
}

// main.nf
include { ALIGN } from './modules/align'
```

Reference: Nextflow modules documentation

### 5.3 Use Subworkflows for Reusable Stages

**Impact: MEDIUM-HIGH (Encapsulates multi-step logic)**

Group related steps into a subworkflow to reuse across pipelines and keep main workflows simple.

**Incorrect (repeat multiple steps in main workflow):**

```nextflow
workflow {
  reads_ch | TRIM | QC
  // ...
}
```

**Correct (define and reuse a subworkflow):**

```nextflow
workflow QC_AND_TRIM {
  take:
    reads_ch
  main:
    reads_ch | TRIM | QC
  emit:
    qc_ch
}

workflow {
  QC_AND_TRIM(reads_ch)
}
```

Reference: Nextflow subworkflow documentation

### 5.4 Prefer Workflow Outputs for Publishing (25.10+)

**Impact: MEDIUM-HIGH (Makes outputs explicit and reduces publishDir misuse)**

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

## 6. Configuration & Profiles

**Impact: MEDIUM-HIGH**

Clear parameter defaults and environment profiles keep deployments consistent.

### 6.1 Provide Parameter Defaults and Validation

**Impact: MEDIUM-HIGH (Prevents silent misconfigurations)**

Define reasonable defaults and validate required params early to fail fast.

**Incorrect (no defaults or validation):**

```nextflow
workflow {
  reads_ch = Channel.fromPath(params.reads)
}
```

**Correct (defaults and validation):**

```nextflow
params.outdir = params.outdir ?: 'results'

if (!params.reads)
  error "Missing required --reads"

workflow {
  reads_ch = Channel.fromPath(params.reads)
}
```

Reference: Nextflow parameters documentation

### 6.2 Split Configuration with includeConfig

**Impact: MEDIUM-HIGH (Keeps config files maintainable)**

Use includeConfig to split base settings, resources, and profiles into dedicated files.

**Incorrect (single monolithic config):**

```nextflow
// nextflow.config
// hundreds of lines here
process.executor = 'slurm'
// ...
```

**Correct (compose configs):**

```nextflow
// nextflow.config
includeConfig 'conf/base.config'
includeConfig 'conf/resources.config'
includeConfig 'conf/profiles.config'
```

Reference: Nextflow configuration include documentation

### 6.3 Keep Secrets Out of Versioned Config

**Impact: MEDIUM-HIGH (Reduces credential leakage)**

Use Nextflow secrets or environment variables rather than hardcoding secrets in config or params. Secrets should not be assigned to pipeline parameters.

**Incorrect (hardcoded secret):**

```nextflow
params.api_key = 'super-secret-token'
```

**Correct (use secrets or env vars):**

```groovy
// nextflow.config
aws {
  accessKey = secrets.MY_ACCESS_KEY
  secretKey = secrets.MY_SECRET_KEY
}
```

**Process usage (inject as env vars):**

```nextflow
process UPLOAD {
  secret 'MY_ACCESS_KEY'
  secret 'MY_SECRET_KEY'

  script:
  """
  upload --access \$MY_ACCESS_KEY --secret \$MY_SECRET_KEY
  """
}
```

Reference: Nextflow secrets documentation

### 6.4 Use Labels and withLabel for Resource Classes

**Impact: MEDIUM-HIGH (Centralizes resource policy and reduces duplication)**

Assign resource classes with process `label` and configure resources centrally using `withLabel` selectors. Use `withName` only for exceptions, and remember `withName` overrides `withLabel`.

**Incorrect (resources scattered in process bodies):**

```nextflow
process ALIGN {
  cpus 16
  memory '32 GB'
  ...
}
```

**Correct (labels + centralized selectors):**

```nextflow
process ALIGN {
  label 'big_mem'
  ...
}

// nextflow.config
process {
  withLabel: big_mem {
    cpus = 16
    memory = 32.GB
    queue = 'long'
  }
}
```

Reference: Nextflow process selectors documentation

## 7. Caching & Provenance

**Impact: MEDIUM-HIGH**

Deterministic outputs and proper reporting preserve provenance and improve reruns.

### 7.1 Encourage -resume with Stable workDir

**Impact: MEDIUM-HIGH (Speeds reruns and saves compute)**

Use a stable workDir and recommend -resume for reruns to reuse cached results.

**Incorrect (ephemeral workDir, no resume guidance):**

```nextflow
workDir = '/tmp/nextflow-work'
```

**Correct (stable workDir for caching):**

```nextflow
workDir = "/scratch/${System.getenv('USER')}/nextflow-work"
```

Reference: Nextflow resume and caching documentation

### 7.2 Keep Outputs Deterministic for Cache Hits

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

### 7.3 Never Modify Input Files In-Place

**Impact: HIGH (Preserves resumability and cache validity)**

Processes that modify their own input files cannot be resumed. Treat inputs as read-only and write outputs to new files.

**Incorrect (mutates input file):**

```nextflow
process CLEAN {
  input:
  path(reads)

  output:
  path(reads)

  script:
  """
  tr -d 'N' < $reads > $reads
  """
}
```

**Correct (write to a new output):**

```nextflow
process CLEAN {
  input:
  path(reads)

  output:
  path('cleaned.fastq')

  script:
  """
  tr -d 'N' < $reads > cleaned.fastq
  """
}
```

Reference: Nextflow cache and resume documentation

### 7.4 Avoid Global Mutable State in Operators

**Impact: MEDIUM-HIGH (Prevents race conditions and nondeterministic cache keys)**

Using implicit global variables inside channel operators can create race conditions and non-deterministic values, which breaks caching. Always declare local variables with `def` inside closures.

**Incorrect (global mutable variable):**

```nextflow
Channel.of(1,2,3).map { v -> X=v; X+=2 }
Channel.of(1,2,3).map { v -> X=v; X*=2 }
```

**Correct (local variable per closure):**

```nextflow
Channel.of(1,2,3).map { v -> def x=v; x+=2 }
Channel.of(1,2,3).map { v -> def x=v; x*=2 }
```

Reference: Nextflow cache and resume documentation

### 7.5 Enable Trace, Report, Timeline, and DAG

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

## 8. Quality & Operations

**Impact: MEDIUM**

Tests, CI profiles, and documentation reduce regressions and operational friction.

### 8.1 Add Automated Pipeline Tests with Small Fixtures

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

### 8.2 Provide a Minimal CI Profile

**Impact: MEDIUM (Makes CI fast and deterministic)**

Define a CI profile with small resources and limited parallelism so tests run quickly and reliably.

**Incorrect (CI uses default heavy profile):**

```nextflow
// CI uses default profile with production resources
```

**Correct (dedicated CI profile):**

```nextflow
profiles {
  ci {
    process.cpus = 1
    process.memory = '1 GB'
    maxForks = 1
    params.small = true
  }
}
```

Reference: Nextflow configuration profiles documentation

### 8.3 Document Inputs, Outputs, and Parameters

**Impact: MEDIUM (Improves usability and supportability)**

Clearly document required inputs, outputs, and parameters in README or parameter files so users run the pipeline correctly.

**Incorrect (params not documented):**

```nextflow
// No parameter documentation for users
```

**Correct (document parameters and defaults):**

```nextflow
// nextflow.config
params {
  reads = null   // input reads glob
  outdir = 'results'
}
```

Reference: Nextflow documentation guidelines

---

## References

1. [https://www.nextflow.io/docs/latest/index.html](https://www.nextflow.io/docs/latest/index.html)
2. [https://www.nextflow.io/docs/latest/dsl2.html](https://www.nextflow.io/docs/latest/dsl2.html)
3. [https://www.nextflow.io/docs/latest/process.html](https://www.nextflow.io/docs/latest/process.html)
4. [https://www.nextflow.io/docs/latest/config.html](https://www.nextflow.io/docs/latest/config.html)
5. [https://www.nextflow.io/docs/latest/channel.html](https://www.nextflow.io/docs/latest/channel.html)
6. [https://www.nextflow.io/docs/latest/cache-and-resume.html](https://www.nextflow.io/docs/latest/cache-and-resume.html)
7. [https://www.nextflow.io/docs/latest/reporting.html](https://www.nextflow.io/docs/latest/reporting.html)
8. [https://www.nextflow.io/docs/latest/secrets.html](https://www.nextflow.io/docs/latest/secrets.html)
9. [https://www.nextflow.io/docs/latest/workflow.html](https://www.nextflow.io/docs/latest/workflow.html)
