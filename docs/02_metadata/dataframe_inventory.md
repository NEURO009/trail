# Dataframe Inventory — IRIS Platform

This document inventories all key dataframes generated across the IRIS analytical pipeline, describing their logical roles, input sources, and output destinations.

---

## 1. Dataframe Catalog

| Dataframe Variable | Notebook Created | Classification | Logical Role & Schema Notes | Upstream Input | Downstream Consumer |
|---|---|---|---|---|---|
| `pos_syndicated_df` | `01_NIELSEN_TRANSFORMATION` | Temporary | Stores raw POS transactions after window-based deduplication (`row_number() == 1`). Columns include `SOURCE_KEY`, `MARKET_KEY`, `PRODUCT_KEY`, `WEEK_ID`, `PROMOTION_KEY`. | `FACT_POS_SYNDICATED` | In-memory joins |
| `nielsen_metrics_df` | `01_NIELSEN_TRANSFORMATION` | Output | Cleaned syndicated POS data with computed ACV percentage variables (`ANY_FEAT_ACV_PCT`, `ANY_DISP_ACV_PCT`). | `pos_syndicated_df`, promotions | `02_IRIS_PRODUCT_DEFINITION` |
| `amazon_metrics_df` | `04_AMAZON_TRANSFORMATION` | Output | Amazon e-commerce POS records mapped to standard Nielsen schemas. Includes baseline updates. | Amazon POS database | `02_IRIS_PRODUCT_DEFINITION` |
| `kroger_metrics_df` | `07_KROGER_STORE_TRANSFORMATION` | Output | Kroger regional/divisional metrics containing synthetic ACV measurements. | Kroger Store POS | `02_IRIS_PRODUCT_DEFINITION` |
| `base_metrics_df` | `02_IRIS_PRODUCT_DEFINITION` | Output | Core UPC records enriched with category configurations, search indices, and availability ratings. Low-volume UPC PPGs replaced with scaled subtotal data. | `nielsen_metrics_df`, `amazon_metrics_df`, `kroger_metrics_df` | `01_BASE_MODEL_FEATURES`, `06_SEASONALITY_FEATURES` |
| `agg_metrics_df` | `02_BASE_MODEL_FEATURE_AGGREGATION` | Output | PPG, Brand, and Category rollups of pricing and distribution features. Includes price decomposition fields. | `base_metrics_df` | `01_MODEL_DATAFRAME`, `04_MODEL_DECOMP` |
| `product_distance_df` | `PRODUCT_DISTANCE_UPLOAD` | Output | Pairwise UPC competitive distance scores normalized to a 0–1 scale. | Circana CSVs, product hierarchy | `03_COMP_PRODUCT_DISTANCING` |
| `comp_mapping_df` | `03_COMP_PRODUCT_DISTANCING` | Output | Maps each product to its ranked list of competitor products by similarity. | `product_distance_df` | `04_COMP_INTERACTION_WEIGHTS` |
| `comp_weights_df` | `04_COMP_INTERACTION_WEIGHTS` | Output | Stores normalized competitive interaction weights per PPG. Weights sum to 1.0 per competitor set. | `comp_mapping_df`, `base_metrics_df` | `05_COMP_INTERACTION_FEATURE_AGGREGATION` |
| `comp_metrics_df` | `05_COMP_INTERACTION_FEATURE_AGGREGATION` | Output | Competitor metrics (price, ACV, display) weighted by interaction weights. | `comp_weights_df`, `base_metrics_df` | `01_MODEL_DATAFRAME`, `04_MODEL_DECOMP` |
| `seasonality_df` | `06_SEASONALITY_FEATURES` | Output | Weekly seasonality indices generated from holiday-configured Prophet models run in a PandasUDF. | `base_metrics_df`, holidays | `01_MODEL_DATAFRAME` |
| `model_features_df` | `01_MODEL_DATAFRAME` | Output | Pivoted wide-format feature matrix. Independent variables are scaled, transformed, and log-adjusted. | `base_metrics_df`, `agg_metrics_df`, `comp_metrics_df`, `seasonality_df` | `02_01_MODEL_EXECUTION_ADF`, `02_02_MODEL_EXECUTION_ADF_NON_BAYESIAN` |
| `coef_raw_df` | `02_01_MODEL_EXECUTION_ADF` | Intermediate | Raw posterior parameters (mean, standard deviation, chains) or point beta estimates. | `model_features_df`, priors | `03_01_COMMIT_MODEL` |
| `coef_history_df` | `03_01_COMMIT_MODEL` | Output | Historical collection of all modeling coefficients across iterations. | `coef_raw_df` | `03_02_MODEL_COEF_DEPLOY` |
| `coef_deployed_df` | `03_02_MODEL_COEF_DEPLOY` | Output | Deployed coefficients. Outliers are replaced using percentile thresholds (P5/P95) and missing records are backfilled. | `coef_history_df` | `04_MODEL_DECOMP`, Scenario Planner, Alerts |
| `decomp_waterfall_df` | `04_MODEL_DECOMP` | Output | Attributed volumes (coefficient * feature) mapped to waterfall order indexes and reconciled against predictions. | `coef_deployed_df`, `model_features_df` | Alerts, `CATEGORY_INCREMENTALITY` |
| `category_incrementality_df` | `CATEGORY_INCREMENTALITY` | Output | Incrementality rates, baseline rates, and cannibalization metrics. | `decomp_waterfall_df` | Scenario Planner |
| `kpi_alert_df` | `31` to `38` Alert Notebooks | Output | Price, frequency, distribution, velocity, or share anomalies matched with volume impact estimates. | `base_metrics_df`, `coef_deployed_df`, `decomp_waterfall_df` | `78-classification-2-3-and-dedup` |
| `classified_alerts_df` | `78-classification-2-3-and-dedup` | Output | Deduplicated alerts grouped by market and category tiers. | `kpi_alert_df` | `80-merge-subs-with-alerts` |
| `subscriber_alerts_df` | `80-merge-subs-with-alerts` | Output | User-personalized alert payloads filtered by brand, retailer, and alert frequency. | `classified_alerts_df`, subscription preferences | `99-logic-app-email` |
