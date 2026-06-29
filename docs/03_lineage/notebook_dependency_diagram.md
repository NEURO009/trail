# Notebook Dependency Diagram — IRIS Platform

This document maps the execution paths and dependency relationships between notebooks within each module of the IRIS Platform.

---

## 1. Notebook Execution Flow Diagrams

### 1.1. Data Transformation & Feature Engineering Flow
```mermaid
graph TD
    %% 01_DATA_TRANSFORMATION
    subgraph Transformation_Module [01_DATA_TRANSFORMATION]
        T01[01_NIELSEN_TRANSFORMATION]
        T03[03_CALENDAR_CREATION]
        T04[04_AMAZON_TRANSFORMATION]
        T05[05_PROFITERO_SEARCH_RANKING]
        T06[06_PROFITERO_AVAILABILITY]
        T07[07_KROGER_STORE_TRANSFORMATION]
        T02[02_IRIS_PRODUCT_DEFINITION]

        T01 --> T02
        T04 --> T02
        T05 --> T02
        T06 --> T02
        T07 --> T02
    end

    %% 02_FEATURE_ENGINEERING
    subgraph Feature_Engineering_Module [02_FEATURE_ENGINEERING]
        FE01[01_BASE_MODEL_FEATURES]
        FE02[02_BASE_MODEL_FEATURE_AGGREGATION]
        FE03[03_COMP_PRODUCT_DISTANCING]
        FE04[04_COMP_INTERACTION_WEIGHTS]
        FE05[05_COMP_INTERACTION_FEATURE_AGGREGATION]
        FE06[06_SEASONALITY_FEATURES]

        T02 -->|IRIS_FACTS_BASE_METRICS.parquet| FE01
        T03 -->|IRIS_MD_CALENDAR.parquet| FE01
        T03 -->|IRIS_MD_HOLIDAYS.parquet| FE06

        FE01 --> FE02
        FE01 --> FE06
        FE01 --> FE04
        
        FE03 --> FE04
        FE04 --> FE05
    end
```

### 1.2. Model Execution Flow
```mermaid
graph TD
    subgraph Model_Execution_Module [03_MODEL_EXECUTION]
        ME01[01_MODEL_DATAFRAME]
        ME02_1[02_01_MODEL_EXECUTION_ADF <br> Bayesian]
        ME02_2[02_02_MODEL_EXECUTION_ADF_NON_BAYESIAN <br> OLS]
        ME03[03_01_COMMIT_MODEL]
        ME04[03_02_MODEL_COEF_DEPLOY]
        ME05[04_MODEL_DECOMP <br> Waterfall]
        ME06[05_MODEL_COEF_EXTRAPOLATION]
        ME07[REVERSE_COEF_FUNC <br> NIELSEN_MARKET_REVERSE_COEF]

        ME01 --> ME02_1
        ME01 --> ME02_2
        
        ME02_1 --> ME03
        ME02_2 --> ME03
        
        ME03 --> ME04
        
        ME04 --> ME05
        ME04 --> ME06
        ME04 --> ME07
    end
```

### 1.3. Proactive Alerts Execution Flow
```mermaid
graph TD
    subgraph Alerts_Module [06_Proactive_Alerts]
        A00[00-main-proactive-alerts <br> Orchestrator]
        A01[01-environment-setup]
        A02[02-05 Import Parquets]
        A20[20-coef-summary]
        
        %% KPIs
        A31[31-process-and-save-base-price]
        A32[32-promo-freq]
        A33[33-acv-distribution]
        A34[34-deepest-promo]
        A35[35-velocity]
        A36[36-share]
        A38[38-display-value]
        
        A78[78-classification-2-3-and-dedup]
        A80[80-merge-subs-with-alerts]
        A99[99-logic-app-email]

        A00 --> A01
        A00 --> A02
        A02 --> A20
        
        %% Parallel ThreadPool triggers
        A20 --> A31
        A20 --> A32
        A20 --> A33
        A20 --> A34
        A20 --> A35
        A20 --> A36
        A20 --> A38
        
        A31 --> A78
        A32 --> A78
        A33 --> A78
        A34 --> A78
        A35 --> A78
        A36 --> A78
        A38 --> A78
        
        A78 --> A80
        A80 --> A99
    end
```

---

## 2. Shared Functions & Execution Patterns

- **`%run` imports**: Downstream notebooks in `01_DATA_TRANSFORMATION`, `02_FEATURE_ENGINEERING`, `03_MODEL_EXECUTION`, and `06_Proactive_Alerts` run `%run ../00_ADMIN/01_IRIS_FUNCTIONS` as their first step to import shared utilities like price smoothing (`stick_smooth`), metrics aggregations, and catalog configurations.
- **`dbutils.notebook.run` calls**: The alert orchestrator `00-main-proactive-alerts` triggers execution steps (setup, imports, classification, email) and parallelizes the 7 individual KPI checkers using python threads.
