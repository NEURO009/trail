# Current Medallion Assessment — IRIS Platform

This document evaluates the existing IRIS platform architecture against industry-standard Medallion Architecture principles (Bronze → Silver → Gold layers).

---

## 1. Compliance Matrix

| Medallion Layer | Ideal Standard | Current IRIS Implementation | Compliance Gap |
|---|---|---|---|
| **Bronze (Raw Ingestion)** | - Raw, immutable source landing.<br>- Schema preservation.<br>- Audit metadata (ingest time, source file).<br>- Append-only write style. | - Raw POS and e-commerce sheets are loaded directly to mounts.<br>- Standard columns are cast immediately upon ingestion.<br>- Absence of structured audit metadata. | **High Gap**: Landing zone lacks schema validation and structured append-only history logs. |
| **Silver (Cleansing & Conformance)** | - Data cleaned, conformed, and standardized.<br>- Consistent deduplication.<br>- Reusable base feature tables. | - Deduplication performed in multiple transformation notebooks via window functions.<br>- Price smoothing applied repeatedly.<br>- Feature datasets are tightly coupled with specific model runs. | **Medium Gap**: Duplicate transformations and business rule logic are computed across notebook modules instead of a unified Silver store. |
| **Gold (Business Ready / Aggregated)** | - Dynamic, aggregated marts.<br>- Ready for consumption by APIs/reporting.<br>- Unified catalogs. | - Deployed coefficients, decompositions, and alerts are stored in custom Parquet folders.<br>- JDBC sync scripts copy data manually to SQL Server and Snowflake database tables. | **Medium Gap**: Lack of managed catalogs, ACID transaction safety, and clean gold marts. |

---

## 2. Key Medallion Violations

1. **Lack of Ingestion Schema Enforcement (Bronze Violation)**:
   - Raw transaction tables like `FACT_POS_SYNDICATED` are consumed directly. If the schema of incoming files changes, downstream processes fail silently or throw compilation errors.
2. **Scattered Deduplication and Smoothing (Silver Violation)**:
   - Deduplication logic (`row_number().over(Window.partitionBy(...).orderBy(desc))) == 1`) is executed independently in both `01_NIELSEN_TRANSFORMATION` and `04_AMAZON_TRANSFORMATION`.
   - Symmetric rolling average price smoothing is applied in both feature engineering and alert generation modules, leading to code duplication.
3. **No Delta Lake Transactions (Platform Violation)**:
   - The platform relies on raw Parquet formats (`.parquet`) for intermediate storage instead of ACID-compliant Delta tables. This risks dirty reads during concurrent read/write executions.
4. **Fragile Workspace Boundaries**:
   - Model variables are read and written across different mounts (`/mnt/iris-nielsen-models`, `/mnt/iris-model-output`) without a unified schema namespace or governance layer.
