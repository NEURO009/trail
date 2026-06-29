# Future-State Medallion Architecture — IRIS Platform

This document outlines the proposed target state architecture for the IRIS platform, implementing Medallion Architecture design patterns (Bronze → Silver → Gold layers) using Delta Lake.

---

## 1. Medallion Layer Design

```
                     [RAW INGESTION LANDING ZONE]
                                  │
                                  ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ BRONZE LAYER (Append-Only Delta Tables)                          │
 │ - Ingest audit metadata: _ingestion_time, _source_file           │
 │ - Preserved schema structure, schema evolution enabled           │
 └────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ SILVER LAYER (Conformed & Cleansed Delta Tables)                 │
 │ - Standardized casting, conformed hierarchies, consistent dedup  │
 │ - Reusable metrics: price per EQ, smoothed prices                │
 │ - Unified features: competitive weights, Prophet seasonality     │
 └────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ GOLD LAYER (Business-Ready / Aggregated Delta Tables)            │
 │ - Model-ready wide datasets                                      │
 │ - Deployed model coefficients, waterfall attributions           │
 │ - Incrementality targets, alert logs                             │
 └──────────────────────────────────────────────────────────────────┘
```

---

## 2. Table Specifications per Layer

### 2.1. Bronze Layer (Raw Ingestion)
- **Tables**: `bronze.nielsen_pos_syndicated`, `bronze.amazon_pos`, `bronze.kroger_pos`, `bronze.profitero_search`, `bronze.profitero_availability`, `bronze.tpm_promotions`.
- **Ingestion Mode**: Append-Only.
- **Audit Columns**:
  - `_ingestion_timestamp`: Current timestamp of data landing.
  - `_source_file_name`: Name of the source file.
  - `_pipeline_run_id`: Execution run identifier from orchestrator.

### 2.2. Silver Layer (Conformed & Cleansed)
- **Tables**:
  - `silver.conformed_nielsen_metrics` — Deduplicated POS scans.
  - `silver.conformed_amazon_metrics` — Standardized Amazon e-commerce sales.
  - `silver.conformed_kroger_metrics` — Divisional Kroger metrics.
  - `silver.product_hierarchy` — Standardized SKU dimensions.
  - `silver.calendar` — Fiscal week date maps.
  - `silver.competitor_weights` — Calculated competitor weights.
  - `silver.seasonality_features` — Prophet seasonality indices.
- **Operations**: Cleansing, null handling, casting, and stick price smoothing UDFs.
- **Format**: Managed Delta Tables with constraint checks (`CHECK (base_price > 0)`).

### 2.3. Gold Layer (Business Ready / Aggregated)
- **Tables**:
  - `gold.model_wide_dataset` — Pivoted wide matrix for model inputs.
  - `gold.deployed_coefficients` — Deployed coefficient tables (outlier-checked).
  - `gold.model_decompositions` — Waterfall attributions.
  - `gold.category_incrementality` — Cannibalization metrics.
  - `gold.kpi_alerts` — personal alert outputs.
- **Optimization**: Delta Liquid Clustering enabled on `market_key` and `product_key` to speed up queries.
