# Data Lineage Diagram — IRIS Platform

This document describes the end-to-end data lineage of the IRIS Platform, tracking datasets from source ingestion to downstream consumption.

---

## 1. End-to-End Data Lineage Flow

```mermaid
graph TD
    %% Source Nodes
    subgraph Raw_Data_Sources [Raw Data Sources]
        src_pos[FACT_POS_SYNDICATED <br> Nielsen POS]
        src_kroger[Kroger Store POS]
        src_amazon[Amazon sales raw]
        src_profitero_search[Profitero search raw]
        src_profitero_oos[Profitero OOS raw]
        src_circana[Circana UPC distance CSVs]
        src_tpm[TPM Promotions raw]
    end

    %% Transformation / Engineering Dataframes
    subgraph Staging_and_Clean_Metrics [Staging & Clean Metrics]
        df_nielsen_metrics[NIELSEN_METRICS.parquet]
        df_amazon_metrics[AMAZON_METRICS.parquet]
        df_kroger_corp[KROGER_TOTAL_CORP_POS.parquet]
        df_base_metrics[IRIS_FACTS_BASE_METRICS.parquet]
        df_sub_metrics[IRIS_FACTS_SUB_METRICS.parquet]
        df_agg_metrics[IRIS_FACTS_AGG_METRICS.parquet]
    end

    subgraph Product_Distancing [Product Competitive Mapping]
        df_ref_distance[IRIS_REF_PRODUCT_DISTANCE.parquet]
        df_comp_mapping[IRIS_REF_COMP_PRODUCT_MAPPING.parquet]
        df_comp_weights[IRIS_REF_COMP_INTERACTION_WEIGHTS]
        df_comp_metrics[IRIS_FACTS_COMP_INTERACTION_METRICS.parquet]
    end

    subgraph Model_Seasonality_and_Execution [Model Seasonality & Execution]
        df_seasonality[IRIS_FACTS_SEASONALITY.parquet]
        df_model_df[MODEL_DATAFRAME.parquet]
        df_coef_raw[MODEL_COEF.parquet <br> Runtime]
        df_coef_history[COEF_HISTORY parquets]
        df_coef_deployed[IRIS_MODEL_COEF_CURRENT]
        df_decomp[MODEL_DECOMP parquets]
    end

    subgraph Alerts_and_Simulations [Alerts & Simulations]
        df_alert_facts[PROACTIVE_ALERTS_FACTS.parquet]
        df_inc_metrics[CATEGORY_INCREMENTALITY.parquet]
    end

    %% Target Interfaces
    subgraph Downstream_Consumption [Presentation & Interfaces]
        ui_scenario_coef[SQL: SCENARIO_MODEL_COEFFICIENTS]
        ui_scenario_fact[SQL: SCENARIO_HISTORICAL_FACTS]
        sf_marts[Snowflake: MART views]
        pbi_dash[Power BI Dashboard]
        email_digest[Logic App Email Digest]
        react_app[React Web Application]
    end

    %% Mapping Lines
    src_pos --> df_nielsen_metrics
    src_amazon --> df_amazon_metrics
    src_kroger --> df_kroger_corp
    
    df_nielsen_metrics --> df_base_metrics
    df_amazon_metrics --> df_base_metrics
    df_kroger_corp --> df_base_metrics
    src_profitero_search --> df_base_metrics
    src_profitero_oos --> df_base_metrics
    
    df_base_metrics --> df_sub_metrics
    df_base_metrics --> df_agg_metrics
    
    src_circana --> df_ref_distance
    df_ref_distance --> df_comp_mapping
    df_comp_mapping & df_base_metrics --> df_comp_weights
    df_comp_weights & df_base_metrics --> df_comp_metrics
    
    df_base_metrics --> df_seasonality
    
    df_base_metrics & df_agg_metrics & df_comp_metrics & df_seasonality --> df_model_df
    
    df_model_df --> df_coef_raw
    df_coef_raw --> df_coef_history
    df_coef_history --> df_coef_deployed
    
    df_coef_deployed & df_model_df --> df_decomp
    df_decomp --> df_inc_metrics
    
    df_coef_deployed & df_base_metrics & df_decomp --> df_alert_facts
    
    %% Target connections
    df_coef_deployed --> ui_scenario_coef
    df_base_metrics --> ui_scenario_fact
    
    df_coef_deployed --> sf_marts
    df_decomp --> sf_marts
    df_inc_metrics --> sf_marts
    
    sf_marts --> pbi_dash
    df_alert_facts --> email_digest
    
    ui_scenario_coef --> react_app
    ui_scenario_fact --> react_app
    sf_marts --> react_app
    
    src_tpm --> df_model_df
```
