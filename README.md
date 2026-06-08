# toph-eval

**A structured taxonomy and evaluation framework for automated root cause analysis of data pipeline failures in enterprise health technology environments.**

Virgo Machine Labs · Version 0.1 · Apache 2.0

---

## Overview

Automated root cause analysis (RCA) for data pipeline failures is a distinct and underserved evaluation category. Existing benchmark frameworks address single-step language tasks; they do not address the multi-source, causal reasoning required to diagnose failures in production ETL pipelines, particularly in regulated environments where clinical coding standards, payer interoperability requirements, and compliance constraints introduce failure modes not present in general-purpose pipeline infrastructure.

This repository provides:

- A structured taxonomy of 63 failure types across 10 categories, derived from observed failure patterns in enterprise health technology pipelines
- A stable code scheme (`CATEGORY_NNN`) for labeling evaluation scenarios and reporting disaggregated results
- A scoring framework defining four independently assessed evaluation dimensions
- A foundation for a reproducible benchmark dataset for systems performing automated pipeline RCA

The taxonomy is the categorical foundation of the Toph Engine evaluation suite. It is published as an open resource to support reproducible evaluation across the research and practitioner community.

---

## Repository Structure

```
toph-eval/
├── README.md           This document
├── TAXONOMY.md         Full taxonomy: 10 categories, 63 failure types
├── CONTRIBUTING.md     Guidelines for submitting additions and corrections
├── LICENSE             Apache 2.0
└── examples/
    └── PERM_001.md     Worked example: write permission denied
```

---

## Taxonomy Summary

| Category | Code Prefix | Failure Types |
|---|---|---|
| Access and Permissions | `PERM` | 6 |
| Authentication and Credentials | `AUTH` | 5 |
| Schema and Data Contract | `SCHEMA` | 7 |
| Data Volume and Quality | `VOLUME` | 6 |
| Connectivity and Infrastructure | `CONN` | 7 |
| Resource Exhaustion | `RESOURCE` | 6 |
| Upstream Dependency Failure | `DEPEND` | 5 |
| Orchestration and Scheduling | `ORCH` | 6 |
| File and Format Errors | `FILE` | 6 |
| Healthcare-Specific | `HEALTH` | 7 |

For the full taxonomy including failure type codes and descriptions, see [`TAXONOMY.md`](./TAXONOMY.md).

---

## Evaluation Dimensions

Each evaluation scenario is scored across four dimensions, assessed independently:

| Dimension | Definition |
|---|---|
| **System identification** | Whether the evaluated system correctly identified the infrastructure component at which the failure manifested |
| **Error classification** | Whether the evaluated system correctly assigned the failure type code from this taxonomy |
| **Causal chain accuracy** | Whether the evaluated system correctly traced the failure to its origin rather than the component at which the error surfaced |
| **Fix correctness** | Whether the recommended remediation correctly addresses the identified root cause |

Causal chain accuracy is accorded the highest evaluative weight. It is the dimension that distinguishes root cause analysis from surface-level error detection.

---

## Scope and Applicability

This taxonomy was developed for enterprise ETL pipelines operating on modern data stacks including Jenkins, Apache Airflow, Databricks, Azure Synapse Analytics, Snowflake, Oracle, Azure Data Lake Storage Gen2, and Amazon S3. The healthcare-specific category (Section 10 of the taxonomy) addresses failure modes arising from clinical coding standards, payer interoperability protocols, and regulatory submission requirements. These failure modes do not appear in general-purpose pipeline monitoring taxonomies.

The framework is intended to be extensible to adjacent regulated verticals including government and public sector data infrastructure. Contributions covering these environments are welcomed.

---

## Versioning

All evaluation results should be reported against a specific taxonomy version number to ensure reproducibility. The current version is `0.1`. See [`TAXONOMY.md`](./TAXONOMY.md) for the full versioning policy.

---

## Contributing

Contributions are accepted via pull request. See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for submission guidelines. Priority areas for contribution are identified in the taxonomy document.

---

## Citation

If you use this taxonomy in published research or evaluation work, please cite it as follows:

```
Sagar, V. (2026). Pipeline Failure Taxonomy for Enterprise Health Technology
Data Pipelines (Version 0.1). Virgo Machine Labs.
https://github.com/virgomachinelabs/toph-eval
```

---

## License

Licensed under the Apache License, Version 2.0. See [`LICENSE`](./LICENSE) for the full terms.

---

*Virgo Machine Labs · virgomachinelabs.com*
