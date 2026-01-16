# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Process Design & Resources (process)

**Impact:** CRITICAL  
**Description:** Correct inputs/outputs and right-sized resources are the largest drivers of pipeline reliability and performance.

## 2. Execution & Scaling (exec)

**Impact:** HIGH  
**Description:** Executor and parallelism tuning prevent overload and shorten time-to-results.

## 3. Dataflow & Metadata (dataflow)

**Impact:** HIGH  
**Description:** Stable channel structure and keyed joins reduce data mismatches and enable scalable streaming.

## 4. Environment & Reproducibility (env)

**Impact:** HIGH  
**Description:** Pinning runtime environments ensures reproducible results across machines and time.

## 5. Workflow Structure & Modularity (structure)

**Impact:** MEDIUM-HIGH  
**Description:** Modular workflows are easier to test, reuse, and maintain as pipelines evolve.

## 6. Configuration & Profiles (config)

**Impact:** MEDIUM-HIGH  
**Description:** Clear parameter defaults and environment profiles keep deployments consistent.

## 7. Caching & Provenance (cache)

**Impact:** MEDIUM-HIGH  
**Description:** Deterministic outputs and proper reporting preserve provenance and improve reruns.

## 8. Quality & Operations (quality)

**Impact:** MEDIUM  
**Description:** Tests, CI profiles, and documentation reduce regressions and operational friction.
