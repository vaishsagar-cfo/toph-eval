# Pipeline Failure Taxonomy for Enterprise Data Pipelines

**Toph Engine Evaluation Framework**  
Virgo Machine Labs

**Version:** 0.1  
**Maintainer:** Virgo Machine Labs  
**License:** Apache 2.0

---

## Abstract

This document presents a structured taxonomy of failure modes in enterprise extract-transform-load (ETL) data pipelines, with particular emphasis on regulated health technology environments. The taxonomy is derived from observed failure patterns in production pipeline infrastructure operating across modern data stacks including Jenkins, Apache Airflow, Databricks, Azure Synapse Analytics, Snowflake, Oracle, Azure Data Lake Storage Gen2, and Amazon S3.

The taxonomy serves as the categorical foundation for the Toph Eval benchmark suite, which provides standardized evaluation scenarios for automated pipeline failure diagnosis systems. Each failure type is assigned a stable identifier used to label evaluation scenarios in the benchmark dataset and to report disaggregated evaluation results.

---

## 1. Introduction

Automated root cause analysis (RCA) for data pipeline failures presents a distinct and underexplored evaluation challenge. Unlike single-step question-answering tasks, pipeline RCA requires a system to reason across multiple heterogeneous log sources simultaneously, construct a causal graph of pipeline execution, and distinguish the origin of a failure from the location at which it manifests. Existing general-purpose evaluation frameworks do not address this task category.

This taxonomy was developed to support rigorous, reproducible evaluation of systems performing automated pipeline RCA. The classification scheme is intended to be exhaustive with respect to failure modes commonly observed in enterprise health technology pipelines, and extensible to adjacent verticals including government and public sector data infrastructure.

A further motivation for this work is the absence of any publicly available failure taxonomy specific to regulated health data pipelines. The healthcare-specific category defined herein (Section 10) captures failure modes arising from clinical coding standards, payer interoperability requirements, and regulatory submission constraints that are not represented in general-purpose pipeline monitoring literature.

---

## 2. Taxonomy Structure

The taxonomy comprises 10 failure categories and 63 failure types. Categories are organized by the proximate mechanism of failure rather than by the system component at which the failure manifests. This design choice reflects the requirements of RCA evaluation: a system that correctly identifies the failing component but misclassifies the mechanism provides only partial diagnostic value.

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

Each failure type is assigned a code of the form `PREFIX_NNN`. These codes are stable across taxonomy versions. Additions increment the minor version number. Removal or renaming of existing codes constitutes a breaking change and increments the major version number. All evaluation results should be reported against a specific taxonomy version to ensure reproducibility.

---

## 3. Failure Categories

### 3.1 Access and Permissions (`PERM`)

This category covers failures in which the pipeline process executes successfully but is denied access to a required resource. The distinguishing characteristic is that the compute layer is functional and the target resource exists; the failure arises from an authorization policy that prevents the operation from completing.

| Code | Description |
|---|---|
| `PERM_001` | Write permission denied on target object (table, container, or storage bucket) |
| `PERM_002` | Read permission denied on source object |
| `PERM_003` | Service principal or service account lacks the required role assignment |
| `PERM_004` | Role-based access control (RBAC) misconfiguration in which the role exists but has not been propagated to the relevant scope |
| `PERM_005` | Cross-subscription or cross-tenant access blocked by policy |
| `PERM_006` | IP allowlist or firewall rule blocking inbound connection from compute resource |

---

### 3.2 Authentication and Credentials (`AUTH`)

This category covers failures in which the pipeline process cannot successfully authenticate to a required service. These failures are distinct from permission failures (Section 3.1) in that the identity presented is not accepted, rather than the identity being accepted but unauthorized.

| Code | Description |
|---|---|
| `AUTH_001` | Expired OAuth token or service principal client secret |
| `AUTH_002` | Expired SSL/TLS certificate on a service endpoint |
| `AUTH_003` | Rotated credential not updated in the relevant Key Vault or secrets manager reference |
| `AUTH_004` | Incorrect credentials supplied due to environment variable misconfiguration |
| `AUTH_005` | Multi-factor authentication or conditional access policy preventing service account authentication |

---

### 3.3 Schema and Data Contract (`SCHEMA`)

This category covers failures arising from a mismatch between the structure of data produced by an upstream system and the structure expected by a downstream consumer. These failures are frequently introduced by schema changes in source systems that are not propagated to dependent pipeline configurations.

| Code | Description |
|---|---|
| `SCHEMA_001` | Column added in upstream schema, absent from target schema |
| `SCHEMA_002` | Column removed from upstream schema, still referenced downstream |
| `SCHEMA_003` | Data type change in upstream schema (e.g., string to integer, nullable to non-nullable) |
| `SCHEMA_004` | Column renamed upstream, breaking the field mapping in the downstream transformation |
| `SCHEMA_005` | Unexpected null values in a field defined as non-nullable |
| `SCHEMA_006` | Date or timestamp format change introduced across source systems |
| `SCHEMA_007` | Character encoding change (e.g., UTF-8 to Latin-1) producing parse failures in text fields |

---

### 3.4 Data Volume and Quality (`VOLUME`)

This category covers failures in which data is successfully extracted and transferred but the content does not meet the expectations of downstream processing. A defining characteristic of this category is that many failures produce no explicit error signal in orchestration logs; the failure manifests only through anomalous row counts or missing field values.

| Code | Description |
|---|---|
| `VOLUME_001` | Zero rows extracted due to an empty result set; no error raised by the pipeline |
| `VOLUME_002` | Significantly fewer rows than expected, indicating a partial extract |
| `VOLUME_003` | Duplicate rows introduced by an upstream source system |
| `VOLUME_004` | Unexpected row count increase attributable to a source system anomaly |
| `VOLUME_005` | Protected health information (PHI) fields present in a pipeline designated as non-PHI (compliance violation) |
| `VOLUME_006` | Required clinical code fields absent from extracted data (ICD-10, CPT, NDC codes) |

---

### 3.5 Connectivity and Infrastructure (`CONN`)

This category covers failures in which the pipeline process cannot establish or maintain a network connection to a required service. These failures may originate in the network layer, the service layer, or in mismatches between the two.

| Code | Description |
|---|---|
| `CONN_001` | Connection refused due to service unavailability or incorrect port configuration |
| `CONN_002` | Connection timeout attributable to network latency or service overload |
| `CONN_003` | DNS resolution failure for a service endpoint |
| `CONN_004` | VPN tunnel or private endpoint misconfiguration |
| `CONN_005` | Database connection pool exhausted under concurrent load |
| `CONN_006` | API rate limit exceeded on a source system (e.g., EHR vendor API, payer API) |
| `CONN_007` | REST API returning a non-2xx status code with no retry or error handling logic in the pipeline |

---

### 3.6 Resource Exhaustion (`RESOURCE`)

This category covers failures in which the pipeline process encounters a hard limit on a computational or storage resource. These failures are distinguished from connectivity failures by the fact that the service is reachable; the failure arises from insufficient capacity.

| Code | Description |
|---|---|
| `RESOURCE_001` | Apache Spark executor out-of-memory (OOM) error |
| `RESOURCE_002` | Query execution timeout on Synapse Analytics, Snowflake, or Oracle |
| `RESOURCE_003` | Disk space exhausted on staging or intermediate storage layer |
| `RESOURCE_004` | Databricks cluster autoscaling upper limit reached under peak load |
| `RESOURCE_005` | Maximum concurrent job limit reached on the pipeline orchestrator |
| `RESOURCE_006` | Storage throughput throttling on ADLS Gen2 or Amazon S3 under high-volume write operations |

---

### 3.7 Upstream Dependency Failure (`DEPEND`)

This category covers cases in which the failing job is not the origin of the failure; rather, a prerequisite job has failed or produced invalid output, rendering the dependent job unable to execute correctly. Accurate diagnosis of failures in this category requires the RCA system to trace the causal chain across multiple jobs and identify the originating failure rather than the proximate one.

This category is considered the most diagnostically demanding in the taxonomy. Surface-level monitoring tools typically report the downstream job as the point of failure because that is where the error surfaces. Correct root cause analysis requires reasoning across the full execution graph, including cases in which the upstream job completes with a success status but produces an empty or invalid result set (silent failure).

| Code | Description |
|---|---|
| `DEPEND_001` | Direct upstream job failed, causing a cascade failure in dependent jobs |
| `DEPEND_002` | Upstream job completed with a success status but wrote zero rows (silent failure) |
| `DEPEND_003` | Upstream job was still executing when the downstream job was triggered due to scheduling misconfiguration |
| `DEPEND_004` | Shared reference table or dimension table was not refreshed prior to execution of the dependent job |
| `DEPEND_005` | Cross-pipeline dependency not represented in the orchestrator DAG (undocumented dependency) |

---

### 3.8 Orchestration and Scheduling (`ORCH`)

This category covers failures attributable to incorrect configuration of the pipeline execution logic, including scheduling, retry behavior, and dependency ordering within the orchestrator.

| Code | Description |
|---|---|
| `ORCH_001` | Job triggered at an incorrect time due to daylight saving time transition or timezone misconfiguration |
| `ORCH_002` | Duplicate execution triggered by overlap between a scheduled run and a manual invocation |
| `ORCH_003` | Retry logic invoking a non-idempotent job, resulting in duplicate record insertion |
| `ORCH_004` | Incorrect dependency ordering in the pipeline directed acyclic graph (DAG) |
| `ORCH_005` | Job execution skipped silently due to a conditional branch evaluating to false |
| `ORCH_006` | Service level agreement (SLA) breach in which the job completed successfully but outside the window required by downstream consumers |

---

### 3.9 File and Format Errors (`FILE`)

This category covers failures in which a data file exists at the expected location but cannot be read or parsed correctly. These failures frequently arise from undocumented changes to upstream file generation processes.

| Code | Description |
|---|---|
| `FILE_001` | Unexpected file format received (e.g., CSV delivered to a pipeline expecting Parquet) |
| `FILE_002` | Corrupted file resulting from an incomplete write or truncated transfer operation |
| `FILE_003` | File not present at the expected path due to a change in the upstream naming convention |
| `FILE_004` | Compression codec mismatch (e.g., gzip-compressed file delivered to a pipeline expecting Snappy) |
| `FILE_005` | Field delimiter change in a delimited text file |
| `FILE_006` | Multi-line field values disrupting row-level parsing |

---

### 3.10 Healthcare-Specific (`HEALTH`)

This category covers failure modes that arise specifically in regulated health data pipeline environments. These failure modes are not represented in general-purpose pipeline monitoring literature and do not appear in taxonomies developed for non-healthcare verticals. Their presence in this taxonomy reflects the requirements of enterprise health technology deployments, where clinical coding standards, interoperability specifications, and regulatory submission constraints introduce failure modes with no analog in other domains.

| Code | Description |
|---|---|
| `HEALTH_001` | ICD-10, CPT, or NDC code format change resulting from annual code set updates |
| `HEALTH_002` | National Provider Identifier (NPI) lookup failure due to an out-of-sync provider directory |
| `HEALTH_003` | Member identifier format change introduced across payer systems |
| `HEALTH_004` | FHIR resource version mismatch (e.g., R3 payload delivered to an R4 consumer) |
| `HEALTH_005` | 834 or 837 EDI transaction set format violation |
| `HEALTH_006` | Centers for Medicare and Medicaid Services (CMS) submission deadline missed due to pipeline execution delay |
| `HEALTH_007` | De-identification process failure resulting in PHI present in a downstream analytics layer |

---

## 4. Evaluation Scoring Dimensions

Evaluation scenarios constructed against this taxonomy are scored across four dimensions, each assessed independently. Reporting all four dimensions separately preserves diagnostic signal that would be lost in a collapsed aggregate score.

| Dimension | Definition |
|---|---|
| **System identification** | Whether the evaluated system correctly identified the infrastructure component at which the failure manifested (e.g., Azure Synapse Analytics vs. Jenkins) |
| **Error classification** | Whether the evaluated system correctly assigned the failure type code from this taxonomy |
| **Causal chain accuracy** | Whether the evaluated system correctly traced the failure to its origin rather than reporting only the component at which the error surfaced |
| **Fix correctness** | Whether the remediation recommended by the evaluated system correctly addresses the identified root cause |

Causal chain accuracy is accorded the highest evaluative weight. It is the dimension that distinguishes root cause analysis from surface-level error detection, and it is the dimension on which general-purpose language models without domain-specific tooling demonstrate the greatest performance deficit.

---

## 5. Versioning Policy

This taxonomy will be updated as new failure patterns are identified in production environments. The versioning policy is as follows:

- **Minor version increment:** Addition of new failure type codes within existing categories, or addition of new categories.
- **Major version increment:** Removal or renaming of existing failure type codes, or restructuring of category boundaries.

All published evaluation results must reference a specific taxonomy version number to ensure reproducibility of benchmark scores.

---

## 6. Contributing

Contributions to this taxonomy are accepted via pull request. Please refer to `CONTRIBUTING.md` for submission guidelines. The following areas are identified as priorities for expansion:

- Additional healthcare-specific failure modes (HEALTH category), particularly those arising from state Medicaid system interfaces
- Government and public sector pipeline failure modes, including those arising from legacy mainframe data extraction
- Snowflake-native and dbt-specific failure patterns
- Streaming pipeline failures arising in Apache Kafka, Amazon Kinesis, and Azure Event Hubs environments

---

*Virgo Machine Labs · virgomachinelabs.com*
