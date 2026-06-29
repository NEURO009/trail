# Gap Analysis — IRIS Platform

This document catalogs structural, process, and architectural gaps in the current IRIS platform, mapping each issue to its architectural impact.

---

## 1. Identified Architectural Gaps

### 1.1. Code and Logic Duplication
- **Gap**: Price smoothing (`stick_smooth`), ACV index checks, and product hierarchy joins are repeated across feature engineering, modeling dataframe preparation, and alerts processing modules.
- **Impact**: Code changes in shared metrics calculations require modifying and testing multiple independent notebooks.

### 1.2. Storage and Data Duplication
- **Gap**: The same metric fields (like sales volume, amount, ACV %) are duplicated across `NIELSEN_METRICS.parquet`, `IRIS_FACTS_BASE_METRICS.parquet`, and intermediate model runtime folders.
- **Impact**: Increased storage footprint and data divergence risks if one parquet file is updated without synchronizing the others.

### 1.3. Fragile Pipeline Orchestration
- **Gap**: Orchestration depends on Azure Data Factory calling chains of notebooks that pass file locations. Inter-pipeline dependencies are tracked via file check scripts instead of integrated task dependencies.
- **Impact**: Difficult debugging, lack of clean failure recovery/retry mechanisms, and pipeline maintenance overhead.

### 1.4. Absence of Schema Governance
- **Gap**: The database layer does not enforce schema constraints for intermediate Parquet files. Schema mutations at source levels break downstream models.
- **Impact**: High risk of runtime exceptions when regression models encounter unexpected columns or mutated types.

### 1.5. Lack of Managed Unity Catalog
- **Gap**: The platform does not use Databricks Unity Catalog. Access permissions are managed at the ADLS storage mount layer using Azure SAS tokens.
- **Impact**: Security governance gap; unable to enforce row-level or column-level access controls inside Databricks, or track detailed data lineage.

---

## 2. Gap Remediation Matrix

| Gap Area | Root Cause | Proposed Medallion Remediation |
|---|---|---|
| **Logic Duplication** | Lack of centralized shared modules. | Standardize calculations into a conformance layer (Silver Delta tables). |
| **Pipeline Fragility** | Reliance on ADF + nested `dbutils.notebook.run`. | Migrate pipeline schedules to Databricks Workflows with task retries. |
| **No Schema Governance** | raw `.parquet` formats used as intermediates. | Implement Delta Lake formats with Schema Enforcement and Evolution enabled. |
| **Access Control Gaps** | Storage mount SAS token authentication. | Implement Unity Catalog catalog-level and row-level RBAC rules. |
