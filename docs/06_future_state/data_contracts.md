# Data Contracts & Quality Metrics — IRIS Platform

This document defines the data contracts, quality constraints, and SLA rules agreed between upstream ingestion systems, Databricks engineering pipelines, and downstream consumption layers.

---

## 1. Upstream Data Ingest Contracts

To prevent source variations from failing modeling pipelines:
- **Syndicated POS Ingest Contracts**:
  - File Format: Parquet.
  - Frequency: Weekly (Monday at 06:00 UTC).
  - Schema Constraints:
    - `SOURCE_KEY` (Decimal, Not Null)
    - `MARKET_KEY` (Decimal, Not Null)
    - `PRODUCT_KEY` (Decimal, Not Null)
    - `WEEK_ID` (Integer, Not Null)
    - `AMOUNT` (Decimal, Not Null, range > 0)
    - `UNITS` (Decimal, Not Null, range >= 0)

---

## 2. Data Quality (DQ) Constraints (Silver/Gold Layers)

We utilize Delta constraints and check expectations to validate table updates:
- **Deduplication Check**: Assert zero duplicate rows for primary keys (`SOURCE_KEY`, `MARKET_KEY`, `PRODUCT_KEY`, `WEEK_ID`).
- **Null Value Enforcement**: Null keys are blocked and routed to quarantine tables (`metadata.quarantined_records`).
- **ACV Percentage Range Checks**:
  - Check: `ANY_FEAT_ACV_PCT BETWEEN 0.0 AND 100.0`.
  - Action: Alerts generated if out-of-bounds metrics exceed 1% of rows.

---

## 3. SLA Matrix

| Pipeline Stage | Operational SLA Target | Dependency | Failure Action |
|---|---|---|---|
| **Raw Ingest (Bronze)** | Monday at 08:00 UTC | Broker feed availability | Trigger alert to Ingest Support team. |
| **Feature Store (Silver)** | Tuesday at 12:00 UTC | Raw ingest completed | Halt execution, trigger failure alerts. |
| **Model Exec (Gold)** | Wednesday at 18:00 UTC | Feature store conformed | Fail pipeline job run, send notification. |
| **SQL Server Uploads** | Wednesday at 22:00 UTC | Model runs finished | Rollback SQL transaction, alert DBA. |
