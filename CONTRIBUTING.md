
# Contributing to toph-eval

Thank you for your interest in contributing to this taxonomy. This document describes the process for submitting additions, corrections, and extensions.

---

## What We Accept

### Additions to existing categories

New failure type codes may be proposed for any existing category. A valid addition must:

- Describe a failure mode that is meaningfully distinct from all existing codes in the category
- Be observable in production pipeline infrastructure (i.e., not purely theoretical)
- Be describable with a single, precise sentence

### New categories

A new category may be proposed if a coherent set of failure modes exists that does not fit within any existing category. A valid new category must contain a minimum of three distinct failure types at the time of proposal.

### Corrections

Corrections to existing descriptions are accepted where the current description is ambiguous, technically imprecise, or incorrect. Please explain the specific problem with the existing description in your pull request.

### Extensions to adjacent verticals

The taxonomy currently prioritizes enterprise health technology environments. Contributions covering the following verticals are actively sought:

- Government and public sector data infrastructure, including legacy mainframe extraction pipelines
- State Medicaid management information system (MMIS) interfaces
- Snowflake-native and dbt-specific failure patterns
- Streaming pipeline failures in Apache Kafka, Amazon Kinesis, and Azure Event Hubs environments

---

## What We Do Not Accept

- Failure modes that are specific to a single vendor implementation and do not generalize across the category
- Duplicate codes that describe the same underlying failure mechanism as an existing code
- Changes that remove or rename existing codes without a corresponding major version increment discussion (open an issue first)

---

## Submission Process

1. Open an issue describing the proposed addition or correction before submitting a pull request. This allows discussion of whether the contribution fits the scope of the taxonomy before implementation work begins.

2. Fork the repository and create a branch named descriptively (e.g., `add-streaming-failures` or `correct-auth-003`).

3. Make your changes to `TAXONOMY.md`. Follow the existing format precisely:
   - Code format: `PREFIX_NNN` where NNN is zero-padded to three digits
   - Description format: a single declarative sentence in the present tense, beginning with a noun phrase describing the failure condition
   - Do not use em dashes

4. Submit a pull request referencing the issue number. Include a brief explanation of the failure mode and at least one example of the infrastructure context in which it has been observed.

---

## Code Assignment

New failure type codes are assigned by the maintainers upon merge. Do not assign your own code numbers in a pull request, as concurrent submissions may create conflicts. Use a placeholder (e.g., `HEALTH_TBD`) in your submission.

---

## Versioning

Accepted additions will be included in the next minor version release. Corrections to descriptions that do not change the meaning of a code may be merged without a version increment. All version changes are recorded in the taxonomy document.

---

## Questions

Open an issue with the label `question` for any queries about scope, process, or taxonomy structure.

---

*Virgo Machine Labs · virgomachinelabs.com*
