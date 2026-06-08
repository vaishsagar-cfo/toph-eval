
# Example Evaluation Scenario: PERM_001

**Failure Type:** Write permission denied on target object  
**Category:** Access and Permissions  
**Code:** `PERM_001`  
**Version:** 0.1

---

## 1. Scenario Description

A nightly extract pipeline transfers claims data from a payer source system through a multi-stage ETL process terminating in a load operation to an Azure Synapse Analytics dedicated SQL pool. The pipeline executes on a scheduled trigger at 02:00 UTC. All upstream stages complete successfully. The load stage fails with a 403 error when the pipeline service principal attempts to insert records into the target table.

The failure is representative of a class of incidents in which a permissions change in the destination environment, such as a role reassignment, a security group modification, or an RBAC policy update, is not reflected in the pipeline service account configuration. The error surfaces at the load stage. The root cause is an authorization policy misconfiguration at the target object level.

**Pipeline:** `nightly-claims-extract`  
**Job ID:** `#841`  
**Execution date:** 2025-05-17  
**Stack:** Jenkins (orchestration), Databricks (transformation), Azure Synapse Analytics (destination), Azure Data Lake Storage Gen2 (staging)

---

## 2. Simulated Log Inputs

The following logs represent the four log sources available to the RCA system for this scenario. All logs are provided simultaneously. The RCA system is not told in advance which log contains the relevant signal.

### 2.1 Jenkins Orchestration Log

```
[2025-05-17 02:14:00] INFO  Pipeline: nightly-claims-extract | Build #841 | Trigger: TIMER
[2025-05-17 02:14:01] INFO  Stage: CONNECT        | Status: SUCCESS | Duration: 4s
[2025-05-17 02:14:07] INFO  Stage: STAGE          | Status: SUCCESS | Duration: 6s
[2025-05-17 02:14:13] INFO  Stage: TRANSFORM      | Status: SUCCESS | Duration: 6s
[2025-05-17 02:14:20] ERROR Stage: LOAD           | Status: FAILURE | Duration: 7s
[2025-05-17 02:14:20] ERROR Build #841 failed at stage LOAD
[2025-05-17 02:14:20] INFO  Finished: FAILURE
```

### 2.2 Databricks Transformation Log

```
[2025-05-17 02:14:08] INFO  Cluster: db-prod-etl-01 | Spark 3.3.2 | Session started
[2025-05-17 02:14:09] INFO  Task T1: ICD-10 normalization          | Status: SUCCESS | Records: 16,303
[2025-05-17 02:14:10] INFO  Task T2: CPT code mapping              | Status: SUCCESS | Records: 16,303
[2025-05-17 02:14:12] INFO  Task T3: NDC normalization             | Status: SUCCESS | Records: 16,303
[2025-05-17 02:14:13] INFO  Output: 16,303 records written to handoff container
[2025-05-17 02:14:13] INFO  Job state: SUCCEEDED
```

### 2.3 Azure Synapse Analytics Load Log

```
[2025-05-17 02:14:14] INFO  Connection: synapse-optum-prod.sql | Status: OK
[2025-05-17 02:14:15] INFO  Authentication: sp-adf-etl-prod | Status: OK
[2025-05-17 02:14:16] INFO  Target: PADbx.dbo.ndc_codes
[2025-05-17 02:14:17] INFO  Schema validation: PASSED
[2025-05-17 02:14:20] ERROR SqlException 403: The INSERT permission was denied
                            on the object 'ndc_codes', database 'PADbx',
                            schema 'dbo'.
[2025-05-17 02:14:20] ERROR Load operation failed. 0 records written.
```

### 2.4 ADLS Gen2 Staging Audit Log

```
[2025-05-17 02:14:07] INFO  File: RD1_icd10.parquet   | Size: 1.8 MB | Status: STAGED
[2025-05-17 02:14:07] INFO  File: RD2_cpt.parquet     | Size: 1.6 MB | Status: STAGED
[2025-05-17 02:14:07] INFO  File: RD3_ndc.parquet     | Size: 1.5 MB | Status: STAGED
[2025-05-17 02:14:07] INFO  Total staged: 4.9 MB | Container: optum-datalake-prod/raw/
[2025-05-17 02:14:21] WARN  Staged files not consumed by load process
[2025-05-17 02:14:21] INFO  Retention policy: 72 hours
```

---

## 3. Ground Truth Labels

The following labels constitute the ground truth for this evaluation scenario. Each label corresponds to one of the four scoring dimensions defined in the taxonomy.

| Dimension | Ground Truth |
|---|---|
| **System identification** | Azure Synapse Analytics |
| **Error classification** | `PERM_001` |
| **Causal chain** | The service principal `sp-adf-etl-prod` authenticated successfully to the Synapse endpoint but does not hold the `db_datawriter` role on `PADbx.dbo`. All upstream stages completed successfully. The Databricks transformation stage produced 16,303 valid records. The staged files remain intact in ADLS Gen2. The failure is isolated to the authorization policy on the target object. |
| **Correct remediation** | Grant the `db_datawriter` role to `sp-adf-etl-prod` on the `PADbx` database in the Synapse Analytics RBAC configuration. Re-execute the load stage. Full pipeline re-execution is not required as staged files remain valid within the 72-hour retention window. |

---

## 4. Scoring Notes

**System identification** is straightforward in this scenario. The Synapse log contains an explicit 403 error. A system that reports Jenkins as the failing component on the basis of the build failure entry in the orchestration log should be scored 0 on this dimension.

**Error classification** requires distinguishing `PERM_001` (write permission denied on target object) from `AUTH_001` (expired credential) and `AUTH_003` (rotated secret not updated). The Synapse log confirms that authentication succeeded (`sp-adf-etl-prod | Status: OK`) before the permission error was raised. A system that misclassifies this as an authentication failure has not correctly interpreted the log sequence.

**Causal chain accuracy** is the primary discriminating dimension in this scenario. The correct answer requires the RCA system to establish that the four upstream stages completed successfully, that 16,303 valid records were produced and staged, and that the failure is isolated to the authorization policy at the target object. A system that reports only the 403 error without establishing the integrity of upstream stages provides an incomplete causal account.

**Fix correctness** is assessed against two criteria: whether the recommended action correctly addresses the identified permission deficit, and whether the system correctly identifies that a full pipeline re-execution is unnecessary given the state of the staged files. A recommendation to re-execute the full pipeline from the beginning should be marked as partially correct.

---

## 5. Common Misclassifications

The following misclassifications have been observed in evaluation runs against this scenario and are documented for reference.

| Misclassification | Incorrect Code | Explanation |
|---|---|---|
| Reporting Jenkins as the failing system | `PERM_001` with incorrect system | The Jenkins log records the build failure but is not the source of the failure. The error originates in Synapse. |
| Classifying as authentication failure | `AUTH_001` or `AUTH_003` | Authentication succeeded. The 403 error is a post-authentication authorization denial, not a credential failure. |
| Recommending full pipeline re-execution | Correct code, incorrect fix | The staged files in ADLS Gen2 are intact and within retention. Only the load stage requires re-execution after the permission is granted. |
| Classifying as `PERM_003` | `PERM_003` | `PERM_003` applies when the service account lacks a required role assignment entirely. In this scenario the service principal has a role assignment but that role does not include write permission on the specific target object. The distinction is subtle but evaluatively significant. |

---

*Virgo Machine Labs · virgomachinelabs.com*
