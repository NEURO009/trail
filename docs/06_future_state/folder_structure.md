# Recommended Workspace Folder Structure — IRIS Platform

This document recommends a standardized Databricks workspace folder directory structure to align with software development lifecycle (SDLC) standards and Medallion Architecture guidelines.

---

## 1. Directory Tree

```
/iris-databricks-workspace/
├── config/                          # Centralized environment parameters
│   ├── env_dev.json
│   ├── env_prod.json
│   └── category_thresholds.json
├── src/                             # Python source packages
│   ├── __init__.py
│   ├── common/                      # Shared utility packages
│   │   ├── __init__.py
│   │   ├── smoothing.py             # stick_smooth algorithms
│   │   ├── connections.py           # JDBC and Key Vault connections
│   │   └── aggregations.py          # Rollup helpers
│   ├── bronze/                      # Bronze ingestion pipelines
│   │   ├── ingest_nielsen.py
│   │   ├── ingest_amazon.py
│   │   └── ingest_kroger.py
│   ├── silver/                      # Silver cleansing & feature engineering
│   │   ├── clean_pos.py
│   │   ├── product_definition.py
│   │   ├── competitor_mapping.py
│   │   ├── competitor_weights.py
│   │   └── seasonality_prophet.py
│   └── gold/                        # Gold modeling & serving
│       ├── model_dataframe.py
│       ├── run_bayesian_model.py
│       ├── run_ols_model.py
│       ├── deploy_coefficients.py
│       ├── model_decomposition.py
│       ├── category_incrementality.py
│       └── proactive_alerts/
│           ├── alerts_orchestrator.py
│           ├── check_kpis.py
│           └── send_logic_app.py
├── tests/                           # Unit and integration test suites
│   ├── unit/
│   └── integration/
├── databricks.yml                   # Databricks Asset Bundle definition file
└── README.md
```

---

## 2. Benefits of the Folder Structure

1. **Decoupled Business Logic**: Utilities are structured into standard python files inside `src/common/` rather than imported using `%run`.
2. **Medallion Segmentation**: Processing directories (`bronze/`, `silver/`, `gold/`) isolate scripts by pipeline stage.
3. **Configuration Isolation**: Environments parameters are saved inside `config/` JSON files, removing hardcoded constants.
4. **CI/CD Compatibility**: The root file `databricks.yml` enables deployments using Databricks Asset Bundles (DAB).
