# Notebook Inventory — IRIS Platform

This document catalog lists all Databricks notebooks running in the IRIS Platform, detailing their functional ownership, input datasets, output tables, and technical operations.

---

## 1. Module: IRIS_CIRCANA_PRODUCT_DISTANCE_MODELS
**Path**: `/IRIS_CIRCANA_PRODUCT_DISTANCE_MODELS/`
- **Notebooks**: 19 category-specific notebooks:
  1. `1_FAMILY_CARE_DRY_TISSUE`
  2. `2_FAMILY_CARE_FACIAL_TISSUE`
  3. `3_BCC_SWIM_DIAPERS`
  4. `4_BCC_TRAINING_PANTS`
  5. `5_BCC_YOUTH_PANTS`
  6. `6_ADULT_CARE_BRIEFS`
  7. `7_ADULT_CARE_GUARDS_SHIELDS`
  8. `8_ADULT_CARE_UNDERPADS`
  9. `9_ADULT_CARE_UNDERWEAR`
  10. `10_ADULT_CARE_PADS`
  11. `11_ADULT_CARE_PANTILINERS`
  12. `12_FEM_CARE_PANTILINERS`
  13. `13_FEM_CARE_TAMPONS`
  14. `14_FEM_CARE_PADS`
  15. `15_BCC_WIPES`
  16. `16_BCC_DIAPERS`
  17. `17_FAMILY_CARE_TOWELS`
  18. `18_FAMILY_CARE_FLUSHABLE_WIPES`
  19. `19_COMMON_DISTANCE_RUNNER` (Loops and orchestrates the category notebooks)
- **Business Purpose**: Establishes competitive distance scores between all UPC pairs in a product category.
- **Technical Operations**:
  - Computes 6-month base prices using window functions over `IRIS_FACTS_BASE_METRICS`.
  - Extracts product attributes from `DIM_UPC_PRODUCT` and casts them to integer fields after regular expression cleaning.
  - Dedups UPC records using row number ordering.
  - Builds feature vectors and feeds them to scikit-learn / XGBoost to predict pairwise similarity.
- **Inputs**:
  - `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_BASE_METRICS.parquet`
  - `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_MD_PRODUCT_HIERARCHY.parquet`
  - `/mnt/iris-nielsen-models/DIM_TABLES2.0/DIM_UPC_PRODUCT.parquet`
  - `/mnt/iris-nielsen-models/DIM_TABLES2.0/DIM_PRODUCT.parquet`
- **Outputs**:
  - `/mnt/iris-model-output/PRODUCT_DISTANCE/CAT_{category_id}_DISTANCE.csv`

---

## 2. Module: IRIS_COEF_INSIGHT
**Path**: `/IRIS_COEF_INSIGHT/`

### 2.1. `01_COEF_INGESTION`
- **Technical Purpose**: Ingests and assigns unique model execution identifiers across the run history.
- **Inputs**: `/mnt/iris-model-output/COEF_HISTORY/*`
- **Outputs**: Temp view `COEF_INGESTED` with sequential `MODEL_ID` mapped to `MODEL_ITERATION_ID` and scope keys.

### 2.2. `02_COEF_EXTRAPOLATION`
- **Technical Purpose**: Detects outlier coefficients and replaces them using volume-weighted median values.
- **Inputs**: Deployed coefficients history.
- **Outputs**: `/mnt/iris-model-output/COEF_EXTRAPOLATED.parquet`

### 2.3. `CONSUMPTION_MODEL_COEF`
- **Technical Purpose**: Prepares and scales coefficient tables for forecasting engines.
- **Inputs**: Deployed model coefficients.
- **Outputs**: `/mnt/iris-model-output/CONSUMPTION_COEF.parquet`

### 2.4. `CONSUMPTION_MODEL_OUTPUT`
- **Technical Purpose**: Generates volume prediction validations (predicted EQ vs. actual EQ).
- **Inputs**: Deployed coefficients, `IRIS_FACTS_BASE_METRICS.parquet`.
- **Outputs**: `/mnt/iris-model-output/CONSUMPTION_PREDICTIONS.parquet`

---

## 3. Module: IRIS_NETFLOW_MODELS
**Path**: `/IRIS_NETFLOW_MODELS/`

### 3.1. `PRODUCT_DISTANCE_UPLOAD`
- **Technical Purpose**: Integrates Circana XGBoost similarity outputs into the master reference directory.
- **Inputs**: `/mnt/iris-model-output/PRODUCT_DISTANCE/*.csv`, `IRIS_MD_PRODUCT_HIERARCHY.parquet`
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_REF_PRODUCT_DISTANCE.parquet`

---

## 4. Module: IRIS_SCENARIO_PLANNER
**Path**: `/IRIS_SCENARIO_PLANNER/`

### 4.1. `01_00_NIELSEN_CROSS_COEF`
- **Technical Purpose**: Rolls up pricing coefficients to brand and category levels using volume weighting.
- **Inputs**: Deployed coefficients where `SOURCE_KEY = 11`.
- **Outputs**: `/mnt/iris-model-output/SCENARIO_PLANNER/NIELSEN_CROSS_COEF.parquet`

### 4.2. `01_01_AMAZON_REVERSE_COEF_FUNC`
- **Technical Purpose**: Inverts competitor interaction weights for Amazon-sourced coefficients (`SOURCE_KEY = 62`).
- **Inputs**: Amazon coefficients, Amazon sales volumes.
- **Outputs**: `/mnt/iris-model-output/SCENARIO_PLANNER/AMAZON_REVERSE_COEF.parquet`

### 4.3. `02_WRITE_COEF`
- **Technical Purpose**: Combines and formats the blended scenarios.
- **Inputs**: Nielsen cross-coefficients and Amazon reverse coefficients.
- **Outputs**: `/mnt/iris-model-output/SCENARIO_PLANNER/BLENDED_COEF.parquet`

### 4.4. `03_UPLOAD_SQL_COEF`
- **Technical Purpose**: JDBC upload helper for scenario coefficients.
- **Inputs**: `/mnt/iris-model-output/SCENARIO_PLANNER/BLENDED_COEF.parquet`
- **Outputs**: Azure SQL Table: `dbo.SCENARIO_MODEL_COEFFICIENTS`

### 4.5. `04_UPLOAD_SQL_FACT`
- **Technical Purpose**: JDBC upload helper for scenario historical facts.
- **Inputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_BASE_METRICS.parquet`
- **Outputs**: Azure SQL Table: `dbo.SCENARIO_HISTORICAL_FACTS`

---

## 5. Module: IRIS_NIELSEN_MODELS / 00_ADMIN
**Path**: `/IRIS_NIELSEN_MODELS/00_ADMIN/`

### 5.1. `00_ENVIRONMENT_INITIATION`
- **Technical Purpose**: Mounts raw and output ADLS containers and establishes Snowflake connection contexts.
- **Inputs**: Key Vault secrets.
- **Outputs**: Mount locations `/mnt/iris-nielsen-models`, `/mnt/iris-kroger-models`, `/mnt/iris-model-output`.

### 5.2. `01_IRIS_FUNCTIONS`
- **Technical Purpose**: Reusable utility functions library. Contains `stick_smooth()`, `ppg_volume_split()`, etc.
- **Inputs**: None.
- **Outputs**: Imported into memory when run via `%run`.

### 5.3. `02_SNOWFLAKE_TO_SQL_SERVER`
- **Technical Purpose**: Reads processed gold tables from Snowflake and writes them to Azure SQL for Web application consumption.
- **Inputs**: Snowflake views.
- **Outputs**: Azure SQL Server tables.

---

## 6. Module: IRIS_NIELSEN_MODELS / 01_DATA_TRANSFORMATION
**Path**: `/IRIS_NIELSEN_MODELS/01_DATA_TRANSFORMATION/`

### 6.1. `01_NIELSEN_TRANSFORMATION`
- **Technical Purpose**: Standardizes raw POS syndicated sales records and deduplicates transactions.
- **Inputs**: `FACT_POS_SYNDICATED`, product and market dimension tables.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/NIELSEN_METRICS.parquet`

### 6.2. `02_IRIS_PRODUCT_DEFINITION`
- **Technical Purpose**: Performs missing volume validation and substitutions.
- **Inputs**: `NIELSEN_METRICS.parquet`, hierarchies, search, and availability tables.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_BASE_METRICS.parquet`

### 6.3. `03_CALENDAR_CREATION`
- **Technical Purpose**: Creates fiscal calendars and holiday mapping tables.
- **Inputs**: `DIM_TIME`, holiday configurations.
- **Outputs**: `IRIS_MD_CALENDAR.parquet`, `IRIS_MD_HOLIDAYS.parquet`

### 6.4. `04_AMAZON_TRANSFORMATION`
- **Technical Purpose**: Formats raw Amazon POS data to match the Nielsen schema.
- **Inputs**: Amazon sales parquets (SOURCE_KEY = 62).
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/AMAZON_METRICS.parquet`

### 6.5. `05_PROFITERO_SEARCH_RANKING`
- **Technical Purpose**: Maps ASIN organic and paid ranking positions to product keys.
- **Inputs**: Raw Profitero search database.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_AMAZON_SEARCH_RANK.parquet`

### 6.6. `06_PROFITERO_AVAILABILITY`
- **Technical Purpose**: Computes weekly availability ratios (1 - OOS rate) per PPG.
- **Inputs**: Raw Profitero availability files.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_AVAILABILITY.parquet`

### 6.7. `07_KROGER_STORE_TRANSFORMATION`
- **Technical Purpose**: Aggregates Kroger store POS transactions to division and regional aggregates.
- **Inputs**: Store POS parquets.
- **Outputs**: `/mnt/iris-kroger-models/KROGER_TOTAL_CORP_POS.parquet`

---

## 7. Module: IRIS_NIELSEN_MODELS / 02_FEATURE_ENGINEERING
**Path**: `/IRIS_NIELSEN_MODELS/02_FEATURE_ENGINEERING/`

### 7.1. `01_BASE_MODEL_FEATURES`
- **Technical Purpose**: Applies ACV thresholds, derives base price per EQ, and filters noise via stick smoothing.
- **Inputs**: Standard metrics parquets.
- **Outputs**: `IRIS_FACTS_BASE_METRICS.parquet` (updated), `IRIS_FACTS_CATEGORY_VELOCITY.parquet`

### 7.2. `02_BASE_MODEL_FEATURE_AGGREGATION`
- **Technical Purpose**: Aggregates features from PPG level to brand and category hierarchies.
- **Inputs**: `IRIS_FACTS_BASE_METRICS.parquet`
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_AGG_METRICS.parquet`

### 7.3. `03_COMP_PRODUCT_DISTANCING`
- **Technical Purpose**: Generates similarity scores based on physical attributes.
- **Inputs**: `IRIS_REF_PRODUCT_DISTANCE.parquet`
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_REF_COMP_PRODUCT_MAPPING.parquet`

### 7.4. `04_COMP_INTERACTION_WEIGHTS` (and `_ADF` variant)
- **Technical Purpose**: Calculates cross-brand competitive interaction weights.
- **Inputs**: Product mapping, base metrics, category configuration.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_REF_COMP_INTERACTION_WEIGHTS/{src}_{cat}.parquet`

### 7.5. `05_COMP_INTERACTION_FEATURE_AGGREGATION` (and `_ADF` variant)
- **Technical Purpose**: Computes weighted competitor features (price, ACV, display).
- **Inputs**: Competitor weights, base metrics.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_COMP_INTERACTION_METRICS.parquet`

### 7.6. `06_SEASONALITY_FEATURES`
- **Technical Purpose**: Trains Prophet models per market/category inside a PandasUDF to compute weekly seasonality indices.
- **Inputs**: `IRIS_FACTS_BASE_METRICS.parquet`, holiday tables.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_FACTS_SEASONALITY.parquet`

---

## 8. Module: IRIS_NIELSEN_MODELS / 03_MODEL_EXECUTION
**Path**: `/IRIS_NIELSEN_MODELS/03_MODEL_EXECUTION/`

### 8.1. `01_MODEL_DATAFRAME`
- **Technical Purpose**: Assembles all features, transforms/log-scales columns, and attaches priors.
- **Inputs**: Base metrics, aggregation metrics, competitor features, seasonality.
- **Outputs**: `/mnt/iris-nielsen-models/MODEL_RUNTIME/MODEL_DATAFRAME/{src}_{cat}.parquet`

### 8.2. `02_01_MODEL_EXECUTION_ADF`
- **Technical Purpose**: Distributes Bayesian regression training (Bambi/PyMC NUTS sampler) inside a PandasUDF.
- **Inputs**: Model wide dataframe, prior coefficients.
- **Outputs**: `/mnt/iris-nielsen-models/MODEL_RUNTIME/MODEL_COEF/`, `/mnt/iris-nielsen-models/MODEL_RUNTIME/MODEL_PRED/`

### 8.3. `02_02_MODEL_EXECUTION_ADF_NON_BAYESIAN`
- **Technical Purpose**: Distributes sign-constrained, Ridge-regularized OLS optimization (SciPy minimize) inside a PandasUDF.
- **Inputs**: Model wide dataframe.
- **Outputs**: `/mnt/iris-nielsen-models/MODEL_RUNTIME/MODEL_COEF/`, `/mnt/iris-nielsen-models/MODEL_RUNTIME/MODEL_PRED/`

### 8.4. `03_01_COMMIT_MODEL`
- **Technical Purpose**: Appends run coefficients to history and deletes temporary runtime paths.
- **Inputs**: Temporary runtime coefficient folders.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_MODEL_OUTPUT/MODEL_COEF/{src}_{group}/{cat}.parquet`

### 8.5. `03_02_MODEL_COEF_DEPLOY`
- **Technical Purpose**: Backfills missing iterations, detects outliers via percentile filters (P5/P95), and computes deployments.
- **Inputs**: Model run history.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_MODEL_COEF_CURRENT/{src}_{cat}.parquet`

### 8.6. `04_MODEL_DECOMP`
- **Technical Purpose**: Computes waterfall contributions per variable (coef * feature_value) and reconciles totals.
- **Inputs**: Deployed coefficients, model input dataframes, competitor interaction metrics.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/MODEL_DECOMP/{src}_{cat}/{iteration}.parquet`

### 8.7. `05_MODEL_COEF_EXTRAPOLATION`
- **Technical Purpose**: Computes higher-level scaling (brand to product) for low-variation parameters.
- **Inputs**: Deployed coefficients, features variation logs.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_DATA_MODEL/IRIS_MODEL_COEF_EXTRAPOLATED.parquet`

### 8.8. `REVERSE_COEF_FUNC` & `NIELSEN_MARKET_REVERSE_COEF_FUNC`
- **Technical Purpose**: Computes cross-elasticity models and market-level coefficients.
- **Inputs**: Deployed coefficients, volume shares.
- **Outputs**: Blended cross-coefficient tables.

---

## 9. Module: IRIS_NIELSEN_MODELS / 04_PARQUET_TO_SNOWFLAKE
**Path**: `/IRIS_NIELSEN_MODELS/04_PARQUET_TO_SNOWFLAKE/`

### 9.1. `01_SCHEMA_CREATION`
- **Technical Purpose**: Introspects Spark schema fields to output SQL DDL commands for Snowflake mapping.
- **Inputs**: Output Parquet files.
- **Outputs**: Snowflake `CREATE TABLE` DDL queries.

---

## 10. Module: IRIS_NIELSEN_MODELS / 05_MODEL_SIMULATIONS
**Path**: `/IRIS_NIELSEN_MODELS/05_MODEL_SIMULATIONS/`

### 10.1. `CATEGORY_INCREMENTALITY`
- **Technical Purpose**: Estimates incremental scan volume, cannibalization ratios, and category lift under alternative scenarios.
- **Inputs**: Decomposition outputs.
- **Outputs**: Incrementality tables.

---

## 11. Module: IRIS_NIELSEN_MODELS / 06_Proactive_Alerts
**Path**: `/IRIS_NIELSEN_MODELS/06_Proactive_Alerts/`

### 11.1. `00-main-proactive-alerts`
- **Technical Purpose**: Master driver that executes KPI checking notebooks in parallel via thread pools.
- **Inputs**: Alert date.
- **Outputs**: Triggers alert compile processes.

### 11.2. `31-process-and-save-base-price`
- **Technical Purpose**: Creates price delta metrics and estimates volume impact.
- **Inputs**: Base metrics, pricing coefficients.
- **Outputs**: Base price alert parquets.

### 11.3. `32-38 KPI Alerts` (Includes `32`, `33`, `34`, `35`, `36`, `38`)
- **Technical Purpose**: Set of notebooks calculating rolling alerts: promo frequency anomalies, ACV distribution slips, deepest discount checks, velocity shocks, dollar share shifts, and display value changes.
- **Inputs**: Base facts, decomposition datasets.
- **Outputs**: Alert records.

### 11.4. `78-classification-2-3-and-dedup`
- **Technical Purpose**: Suppresses duplicates within sending cadences and groups records by market/category tiers.
- **Inputs**: Output KPI alert tables.
- **Outputs**: Master alert dataset.

### 11.5. `80-merge-subs-with-alerts`
- **Technical Purpose**: Joins subscriber profiles with active alerts based on market, category, and brand filters.
- **Inputs**: Subscriber rules config, master alerts.
- **Outputs**: Subscriber alert payloads.

### 11.6. `99-logic-app-email`
- **Technical Purpose**: Compiles HTML content summaries and posts payloads to Azure Logic Apps HTTP endpoints.
- **Inputs**: Subscriber payloads.
- **Outputs**: HTTP status calls.

---

## 12. Module: IRIS_TPO
**Path**: `/IRIS_TPO/`

### 12.1. `01_FEATURE_ENGINEERING`
- **Technical Purpose**: Joins planned promotion calendar flags, trade spend, and depths.
- **Inputs**: Raw TPM (Trade Promotion Management) databases.
- **Outputs**: `/mnt/iris-nielsen-models/IRIS_TPO/TPO_FEATURES.parquet`

### 12.2. `02_EVENT_REALIGNMENT_MODEL`
- **Technical Purpose**: Align planned TPM event calendars with actual observed promotion scans from Nielsen/Kroger.
- **Inputs**: Planned promotions, actual POS metrics.
- **Outputs**: Realignment models.
