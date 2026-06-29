# ADR 001: Medallion Architecture with Delta Lake

- **Status**: Approved.
- **Date**: 2026-06-29
- **Author**: Principal Enterprise Data Architect

---

## 1. Context and Problem Statement

The existing IRIS Platform stores intermediate and final results in raw Parquet format (`.parquet`) across various ADLS mount storage locations. Because Parquet format does not support ACID transactions or schema enforcement:
- Concurrent read/write runs risk reading dirty files.
- Upstream schema updates pass through silent checks and cause downstream model run failures.
- Duplicate metrics cleaning and calculations are executed in parallel pipelines, creating duplicate files and inconsistent outputs.

---

## 2. Proposed Decision

We will reorganize the storage layer into a managed Delta Lake implementing the Medallion Architecture pattern:
1. **Bronze Schema**: Immutable append-only raw tables preserving original columns and adding ingestion timestamps.
2. **Silver Schema**: Cleansed, deduplicated, and conformed features tables. Centralizes pricing calculation and price smoothing logic.
3. **Gold Schema**: Analytical models wide dataframes, deployed coefficients, and alerting outputs optimized using Delta Liquid Clustering.

---

## 3. Consequences

### 3.1. Positive Outcomes
- **Data Consistency**: Price calculations, deduplication, and smoothing logic are conformed in Silver tables, eliminating duplication.
- **Pipeline Stability**: Delta schema enforcement blocks faulty database writes, preventing pipeline failures.
- **Query Performance**: Liquid clustering on Gold tables speeds up scenario planner API lookups.
- **Audit Lineage**: Delta time travel log allows auditing historic model coefficient runs.

### 3.2. Trade-offs & Negatives
- **Migration Effort**: Re-writing paths in notebooks from Parquet format to Delta tables requires refactoring code and schema adjustments.
- **Transactional Overhead**: Small updates incur compute latency due to transaction logs updates.
