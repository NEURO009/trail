# Orchestration using Databricks Workflows — IRIS Platform

This document describes the design configurations for migrating pipeline orchestration from Azure Data Factory (ADF) to native Databricks Workflows.

---

## 1. Pipeline DAG Workflows Architecture

Migrating to Databricks Workflows removes the dependency on nested `dbutils.notebook.run` calls and establishes a unified Directed Acyclic Graph (DAG) for processing.

```
       [ingest-raw-nielsen] ──────────► [clean-nielsen-metrics]
                                                │
                                                ▼
     [verify-product-definitions] ◄───── [clean-amazon-metrics]
                  │
                  ├───► [calculate-base-features]
                  │              │
                  │              ├───────────────┐
                  │              ▼               ▼
                  │     [calculate-weights] [train-seasonality]
                  │              │               │
                  │              ▼               │
                  │     [aggregate-comp-metrics] │
                  │              │               │
                  ▼              ▼               ▼
            [assemble-wide-model-dataframe] ◄─────┘
                  │
                  ▼
     ┌────────────┴────────────┐
     ▼                         ▼
[run-bayesian-model]      [run-ols-model]
     │                         │
     └────────────┬────────────┘
                  ▼
          [commit-model-runs]
                  │
                  ▼
          [deploy-coefficients]
                  │
                  ├──────────────────────────────┐
                  ▼                              ▼
          [calculate-decomp]            [upload-to-sql-server]
                  │                              │
                  ▼                              ▼
          [sync-to-snowflake]            [run-proactive-alerts]
```

---

## 2. Task Parameters & Retry Policies

- **Retries**: Each task is configured with:
  - `max_retries`: 2.
  - `min_retry_interval_millis`: 60,000 (1 minute).
  - `retry_on_timeout`: True.
- **Compute Cluster Sharing**:
  - Tasks utilize a shared **New Job Cluster** to minimize cluster startup time overhead.
  - Cluster Profile: Databricks Runtime 14.3 LTS ML, auto-scaling from 2 to 8 worker nodes.

---

## 3. Alerts and Monitoring

- **Email Notifications**: Alerts are triggered on `Failure` or `SLA Exceeded` events.
- **Slack / Teams Integration**: Webhook notification channels are set up to alert engineering teams immediately if critical tasks fail.
