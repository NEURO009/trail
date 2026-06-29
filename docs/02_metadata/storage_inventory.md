# Storage Inventory — IRIS Platform

This document inventories the storage layers, folder hierarchies, databases, and schemas used by the IRIS pipeline.

---

## 1. Storage Layers Overview

The IRIS platform stores and reads data across three primary enterprise storage systems:
1. **Azure Data Lake Storage Gen2 (ADLS Gen2)**: Primary landing, staging, and feature store layer. Files are formatted as Parquet or Delta, organized in folders, and mounted via Databricks mounts.
2. **Snowflake Enterprise Data Warehouse**: Serves as the high-performance analytics layer for business intelligence (Power BI) and historical analytics.
3. **Azure SQL Database**: Stores transactional application metadata, user configurations, and scenario planner inputs/outputs.

---

## 2. ADLS Gen2 Folder Hierarchy & Paths

All storage paths are mounted via DBFS (Databricks File System) under the following mounts:
- `/mnt/iris-nielsen-models` — Syndicated POS data and model input feature store.
- `/mnt/iris-kroger-models` — Regional Kroger store data storage.
- `/mnt/iris-model-output` — Model coefficient history and simulation directories.

### 2.1. Ingestion & Transformation Paths (`/mnt/iris-nielsen-models/`)
- `/FACT_TABLES2.0/FACT_POS_SYNDICATED/` — Raw syndicated POS data files ingested from external brokers.
- `/DIM_TABLES2.0/` — Location of dimensions files: `DIM_PRODUCT.parquet`, `DIM_MARKET.parquet`, `DIM_TIME.parquet`, and `DIM_UPC_PRODUCT.parquet`.
- `/IRIS_DATA_MODEL/` — Master metrics and feature store dataframes:
  - `NIELSEN_METRICS.parquet` — Normalized POS metrics.
  - `AMAZON_METRICS.parquet` — Normalized Amazon sales.
  - `IRIS_MD_PRODUCT_HIERARCHY.parquet` — Consolidated SKU hierarchy.
  - `IRIS_MD_CALENDAR.parquet` — Retail 4-5-4 calendar maps.
  - `IRIS_MD_HOLIDAYS.parquet` — Holiday markers.
  - `IRIS_FACTS_BASE_METRICS.parquet` — Enriched baseline metrics.
  - `IRIS_FACTS_SUB_METRICS.parquet` — Backup subtotal POS.
  - `IRIS_FACTS_AGG_METRICS.parquet` — Brand/Category aggregate metrics.
  - `IRIS_REF_PRODUCT_DISTANCE.parquet` — Pairwise UPC distance scores.
  - `IRIS_REF_COMP_PRODUCT_MAPPING.parquet` — Mapped competitive items.
  - `IRIS_REF_COMP_INTERACTION_WEIGHTS/` — Extracted competitor weights.
  - `IRIS_FACTS_COMP_INTERACTION_METRICS.parquet` — Aggregated competitor metrics.
  - `IRIS_FACTS_SEASONALITY.parquet` — Prophet seasonality indices.

### 2.2. Kroger Regional POS Paths (`/mnt/iris-kroger-models/`)
- `/KROGER_STORE_POS/` — Raw store-level transacted records.
- `KROGER_TOTAL_CORP_POS.parquet` — Aggregated divisional and corporate records.

### 2.3. Model Output & History Paths (`/mnt/iris-model-output/`)
- `/MODEL_RUNTIME/` — Temporary directory for active Spark runs:
  - `MODEL_DATAFRAME/` — Pivoted feature frames.
  - `MODEL_COEF/` — Fitted coefficients.
  - `MODEL_PRED/` — Predictions.
- `/COEF_HISTORY/` — Historical coefficients database.
- `/IRIS_DATA_MODEL/IRIS_MODEL_COEF_CURRENT/` — Current deployed model coefficients.
- `/IRIS_DATA_MODEL/MODEL_DECOMP/` — Waterfall decomposition parquets.
- `/PRODUCT_DISTANCE/` — Raw similarity CSVs output by XGBoost models.

---

## 3. Snowflake Warehouse Schema

Snowflake is partitioned into staging and analytics marts:
- **Database: `IRIS_ANALYTICS`**:
  - **Schema: `STAGING`**: Ingests direct exports from ADLS parquets via Snowflake integration pipelines.
  - **Schema: `MART`**: Relational analytics reporting tables transformed and materialized via DBT models.
- **Database: `TPM_NA_SECURE`**:
  - Highly secure, isolated database containing Trade Promotion Management schedules, spend allocations, and actual events.

---

## 4. Azure SQL Database Inventory

Relational SQL Server database partitioned by function:
- **Database: `iris-backend-db`**:
  - **Schema: `dbo`**:
    - `dbo.SCENARIO_MODEL_COEFFICIENTS` — Blended scenario planning inputs.
    - `dbo.SCENARIO_HISTORICAL_FACTS` — Historical facts database.
  - **Schema: `security`**:
    - `security.USERS` — User master directory.
    - `security.RBAC_MAPPING` — Map AD Group -> Tools/Reports.
    - `security.PEA_USER_SECURITY` — RLS lookup table mapping User -> L4 (Markets) + L7 (Categories).
