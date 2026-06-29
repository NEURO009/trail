# Technical Debt Assessment — IRIS Platform

This document identifies, ranks, and prioritizes technical debt items within the IRIS platform codebase and architecture.

---

## 1. Technical Debt Register

| Debt Item | Description | Severity | Remediation Effort | Priority |
|---|---|---|---|---|
| **Hardcoded Constants** | Configuration properties (like ACV thresholds, holiday windows, and outlier bounds) are hardcoded inside individual notebooks. | **High** | Low | **High** |
| **Fragile `%run` Imports** | Notebooks depend on `%run ../00_ADMIN/01_IRIS_FUNCTIONS` to load functions into memory, polluting the global namespace. | **Medium** | Medium | **Medium** |
| **Manual JDBC Writes** | Hand-rolled JDBC connection loops write data to Azure SQL and Snowflake, lacking transaction rollback support. | **High** | Medium | **High** |
| **Python ThreadPool Alerting** | Proactive alerts run on a single driver node using python threads (`ThreadPool`) to trigger parallel sub-notebook runs. | **Medium** | High | **Medium** |
| **Untracked Model Artifacts** | PyMC/Bambi Bayesian posterior model files are deleted after run completion, retaining only raw coefficient averages. | **Medium** | Medium | **Low** |

---

## 2. Detailed Debt Remediation Guidelines

### 2.1. Centralizing Configurations
- **Current State**: `01_BASE_MODEL_FEATURES` and `31-process-and-save-base-price` contain hardcoded arrays of product categories and threshold parameters.
- **Future State**: Move configuration parameters to a managed metadata table in Unity Catalog (`metadata.category_configurations`). Load configs dynamically via SQL query in helper scripts.

### 2.2. Migrating `%run` to Python Libraries
- **Current State**: Code is imported via Databricks `%run` command, which limits IDE auto-completion and code quality checks.
- **Future State**: Package helper utilities into a structured Python package (`iris_core`) and install it onto Databricks clusters.

### 2.3. Standardizing JDBC Operations
- **Current State**: Notebooks execute SQL strings to truncate target tables before uploading rows.
- **Future State**: Replace custom JDBC notebooks with structured Spark write processes (`spark.write.format("sqlserver")` or standard Delta-to-Snowflake connectors).

### 2.4. Alerting Modernization
- **Current State**: Single-driver ThreadPool runs risk node failure and out-of-memory errors.
- **Future State**: Re-write KPI check processes to leverage Spark's parallel map/UDF execution engine, distributing checks across the cluster.
