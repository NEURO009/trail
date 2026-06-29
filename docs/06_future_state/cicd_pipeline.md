# CI/CD Pipeline using Databricks Asset Bundles — IRIS Platform

This document describes the continuous integration and continuous deployment (CI/CD) pipelines designed to automate deployments for Databricks notebooks, Python packages, and DBT configurations.

---

## 1. Branching Strategy and Workflows

Development teams follow standard Git workflows:
- **`feature/*`**: Short-lived branches for feature additions. Merge requires PR approvals and passing unit tests.
- **`develop`**: Automated deploy to DEV workspace. Runs integration test suites.
- **`main`**: Target release branch. Automated deploy to production secure environment using Databricks Asset Bundles.

---

## 2. CI/CD Architecture (Azure DevOps Pipelines)

```
[Developer Push] ──► [Azure Pipeline Trigger]
                           │
                           ▼
                 [STEP 1: Run PyTest] ──► (unit test PySpark functions)
                           │
                           ▼
                 [STEP 2: Bundle Assets] ──► (DAB build compilation)
                           │
                           ▼
                 [STEP 3: Deploy DAB] ──► (push workspace folders to Azure)
                           │
                           ▼
                 [STEP 4: Run DBT Deploy] ──► (run dbt migrations on Snowflake)
```

### 2.1. Databricks Asset Bundle (DAB) Configuration
The project uses `databricks.yml` to define resources:
```yaml
bundle:
  name: iris-analytics

targets:
  dev:
    workspace:
      host: https://adb-dev.azuredatabricks.net
      root_path: /Shared/dbx/iris-dev
  prod:
    workspace:
      host: https://adb-prod.azuredatabricks.net
      root_path: /Shared/dbx/iris-prod

resources:
  jobs:
    nielsen_model_run:
      name: "[Prod] IRIS Nielsen Model Execution Pipeline"
      tasks:
        - task_key: model_df
          notebook_task:
            notebook_path: ./src/gold/model_dataframe.py
```

---

## 3. Automated Test Verification
- **PyTest Unit Tests**: Run on Azure DevOps build agents. Mocks Spark sessions to validate stick smoothing UDF logic and competitor weight allocation calculations.
- **DBT Data Tests**: Run post-deployment to Snowflake using `dbt test` to check column constraints and primary keys.
