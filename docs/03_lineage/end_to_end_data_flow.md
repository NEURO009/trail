# End-to-End Data Flow — IRIS Platform

This document describes the step-by-step data flow progression across the IRIS analytical processing pipeline, tracing data from raw landing containers to downstream applications.

---

## 1. Pipeline Stage Execution Checklist

```
 [STAGE 1: INGESTION]
   │
   ├─► Nielsen POS scanner files, Product configurations, Market definitions
   ├─► Amazon POS sales parquets (SOURCE_KEY = 62)
   ├─► Kroger Store transaction tables (division/division level)
   ├─► Profitero keyword rankings & out-of-stock daily files
   └─► Circana product similarity matrix CSV sheets
   │
 [STAGE 2: STAGING & CLEANING]
   │
   ├─► Ingested records parsed via PySpark
   ├─► Window-based deduplication removes duplicate keys (keeps row_number = 1)
   ├─► Columns standardized to uniform types (decimal mappings)
   └─► Outputs: NIELSEN_METRICS.parquet, AMAZON_METRICS.parquet, KROGER_TOTAL_CORP_POS.parquet
   │
 [STAGE 3: FEATURE ENGINEERING]
   │
   ├─► Base metrics calculation: Price per EQ = AMOUNT / EQ
   ├─► Stick algorithm smoothing: 3-week symmetric moving average removes spikes
   ├─► Competitor set weights: Similarity indexes combined with volume shares (sum = 1.0)
   ├─► Competitor features: Weighted price, ACV, display variables compiled
   ├─► Prophet seasonality: Distributed PandasUDF trains models with holiday maps
   └─► Output facts: IRIS_FACTS_BASE_METRICS.parquet, IRIS_FACTS_COMP_INTERACTION_METRICS.parquet, IRIS_FACTS_SEASONALITY.parquet
   │
 [STAGE 4: MODELLING RUNTIME PREPARATION]
   │
   ├─► Joins: Base facts, Brand aggregations, Competitor metrics, Seasonality
   ├─► Filter: ACV distribution threshold limits qualifying product-market combinations
   ├─► Scale: log(1 + x) transformations applied to features
   ├─► Priors: Previous run parameters joined as prior normal mean bounds
   └─► Output: Wide-format parquet written to temporary MODEL_RUNTIME/ paths
   │
 [STAGE 5: MODEL TRAINING & COMMITTING]
   │
   ├─► Bayesian fit: Distributed PandasUDF triggers Bambi/PyMC MCMC sampling
   ├─► OLS fit: Constrained L-BFGS-B minimizes squared errors (enforces sign direction)
   ├─► Committing: Runs appended to COEF_HISTORY parquets; runtime directories cleared
   └─► Deployment: Backfills missing combinations and overrides outliers via P5/P95 percentile filters
   │
 [STAGE 6: ATTRIBUTION & SIMULATIONS]
   │
   ├─► Decompositions: waterfall attribution calculates contribution (coefficient * delta)
   ├─► Incrementality: Category aggregations check cannibalization vs true category lift
   └─► Scenario Planner: Blends Amazon/Nielsen coefficients and aggregates cross-coefficients
   │
 [STAGE 7: SYNCHRONIZATION]
   │
   ├─► Snowflake export: Spark Snowflake JDBC driver updates staging tables
   ├─► DBT compiler: Updates Gold reporting database views
   └─► SQL Server export: Pushes blended scenario and historical facts via JDBC connectors
   │
 [STAGE 8: PRESENTATION & CONSUMPTION]
   │
   ├─► Power BI dashboards display model health statistics directly from Snowflake
   ├─► Scenario planning simulator fetches parameters from SQL Server to display charts
   └─► Parallel alert threads monitor indicators, match subscribers, and trigger Logic Apps
```
