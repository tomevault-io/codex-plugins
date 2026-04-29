# NOAA Climate Data Publication System

## Purpose

This repository produces **publication-grade scientific data artifacts** derived from the NOAA Integrated Surface Database (ISD) / Global Hourly dataset.

The system is not primarily an analysis pipeline.

It is a **specification-constrained data publication system** designed to generate:

* canonical cleaned datasets
* domain-specific datasets
* quality and usability evidence
* reproducible release manifests

All agents working in this repository must prioritize **artifact integrity, provenance, and reproducibility**.

---

# 1. Core System Model

The system transforms raw NOAA observations into publishable research artifacts.

```
NOAA raw files
        ↓
specification‑constrained parser/cleaner
        ↓
canonical dataset
        ↓
domain datasets
        ↓
quality evidence artifacts
        ↓
release manifests
```

Each stage must preserve lineage so every published artifact can be traced back to its original NOAA source data.

---

# 2. Repository Responsibilities

This repository is responsible for producing deterministic scientific artifacts:

### Canonical Dataset

The canonical dataset is the **cleaned observation-level dataset** preserving NOAA semantics.

Properties:

* no sentinel values
* explicit null semantics
* preserved quality codes
* deterministic schema
* stable column naming

The canonical dataset is the **foundation artifact** from which all other outputs derive.

---

### Domain Datasets

Domain datasets project the canonical dataset into reusable scientific domains.

Examples:

* core_meteorology
* wind
* precipitation
* clouds_visibility
* pressure_temperature
* remarks

Domain datasets must:

* maintain stable schemas
* remain joinable across domains
* avoid embedded aggregation logic

Researchers must be able to compose their own analyses using these datasets.

---

### Quality Evidence Artifacts

Quality artifacts quantify how the cleaning system affects the data.

Examples include:

* field_completeness
* sentinel_frequency
* quality_code_exclusions
* domain_usability_summary
* station_year_quality

These artifacts are **first‑class outputs**, not debugging tools.

They provide transparency for scientific use.

---

### Release Manifests

Each build must generate a deterministic manifest describing all produced artifacts.

Required metadata fields:

* artifact_id
* schema_version
* build_id
* input_lineage
* row_count
* checksum
* creation_timestamp

Release manifests enable reproducible scientific data releases.

---

# 3. Specification‑Driven Cleaning

The parser and cleaning system enforce rules derived from the NOAA ISD documentation.

Rule types include:

* sentinel handling
* domain validation
* range validation
* token width enforcement
* field arity constraints
* allowed quality codes

Agents must treat the NOAA documentation as the **authoritative reference**.

Cleaning behavior must be traceable to specification rules or explicitly documented engineering safeguards.

---

# 4. Rule Provenance and Governance

Every enforced rule must have traceable provenance.

Rule classes include:

* documented_exact
* documented_inferred
* engineering_guard
* legacy_behavior

Rules that are stricter than documentation should not silently destroy data.

Instead, agents should prefer:

* flagging observations
* recording quality evidence
* preserving original values when scientifically appropriate

This separation prevents the pipeline from introducing unintended scientific bias.

---

# 5. Artifact Reproducibility Requirements

All artifacts produced by the system must be deterministic.

Identical inputs and configuration must produce identical outputs.

Agents must ensure:

* stable serialization formats
* deterministic row ordering
* consistent null representations
* reproducible checksums

The release layout must follow:

```
release/
  build_<build_id>/
    canonical_cleaned/
    domains/
    quality_reports/
    manifests/
```

This structure forms the **publication contract** of the system.

---

# 6. Dataset Contracts

Schemas define the public interface of every artifact.

Agents must treat schemas as **versioned contracts**.

Required contract properties:

* stable column names
* explicit null semantics
* consistent data types
* documented join keys

Breaking schema compatibility must follow an explicit version change.

---

# 7. Quality Evidence Philosophy

Cleaning decisions must remain scientifically transparent.

Agents should avoid silently removing or modifying observations when documentation support is weak.

Prefer generating evidence such as:

* nullification counts
* QC exclusion statistics
* sentinel frequency metrics

These artifacts allow researchers to evaluate how cleaning affects the dataset.

---

# 8. Development Priorities

Current architectural priorities are:

1. Define and freeze artifact contracts.
2. Implement deterministic release builds.
3. Publish domain dataset registry modules.
4. Generate reproducible quality evidence artifacts.
5. Harden CI validation for schemas, lineage, and reproducibility.

The existing parser and rule coverage foundation should remain stable.

---

# 9. Constraints for Automated Agents

Agents modifying this repository must follow these constraints.

### Preserve Artifact Contracts

Do not change schema definitions without explicit version updates.

### Maintain Determinism

Changes must not introduce nondeterministic output behavior.

### Preserve Data Transparency

Avoid transformations that hide or discard information without producing evidence artifacts.

### Protect Rule Provenance

Every rule must remain traceable to documentation or explicit engineering rationale.

---

# 10. Conceptual Model

This repository implements a **specification‑constrained scientific data publication framework**.

```
NOAA specification
        ↓
rule extraction
        ↓
spec‑driven parser/cleaner
        ↓
canonical dataset
        ↓
domain datasets
        ↓
quality evidence
        ↓
reproducible release artifacts
```

The ultimate objective is to produce **transparent, reproducible climate data products suitable for scientific research and citation.**

---
> Source: [balajikesavan90/noaa-spec](https://github.com/balajikesavan90/noaa-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
