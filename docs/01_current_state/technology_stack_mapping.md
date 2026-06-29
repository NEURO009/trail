# Technology Stack Mapping — IRIS Platform

This document catalogs and maps all components of the technology stack utilized in the IRIS Platform, showing their roles, layers, and dependencies.

---

## 1. Technology Stack Inventory

| Component | Technology | Primary Role in IRIS | Layer | Dependencies |
|---|---|---|---|---|
| **Pipeline Orchestration** | Azure Data Factory (ADF) | Schedules and triggers POS data ingestion, metrics aggregation, feature engineering, and model training pipelines. | Orchestration | Azure Databricks, Azure SQL Server, Snowflake |
| **Alert Orchestration** | Azure Logic Apps | Triggered via HTTP POST to orchestrate subscriber alert email delivery. | Orchestration / Integration | Azure Functions, SMTP / Mail Gateway |
| **Core Compute & Processing** | Apache Spark / PySpark | Distributed computation engine for POS data cleaning, joins, aggregations, feature engineering, and pipeline execution. | Compute | Azure Blob Storage Gen2 |
| **Bayesian Regression** | Bambi / PyMC / ArviZ / Xarray | Bayesian regression model training via Markov Chain Monte Carlo (MCMC) NUTS sampling to estimate coefficients. | Machine Learning / Analytics | Python, PySpark (PandasUDF) |
| **Frequentist Regression** | SciPy (L-BFGS-B optimizer) | Solves Ridge-regularized, sign-constrained Ordinary Least Squares (OLS) models. | Machine Learning / Analytics | Python, PySpark (PandasUDF) |
| **Product Similarity Modeling** | XGBoost / scikit-learn / VectorAssembler | Trains distance models to output pairwise product competitive similarity scores. | Machine Learning / Analytics | Python, PySpark |
| **Seasonality Modeling** | Prophet (by Meta) | Computes holiday-adjusted weekly seasonality indices using yearly and monthly Fourier components. | Machine Learning / Analytics | Python, PySpark (PandasUDF) |
| **Data Warehousing & Sync** | Snowflake | Gold layer storage containing IRIS Model Data and Trade Promotion Management (TPM) secure databases. | Storage / Database | Snowflake-Spark Connector, DBT |
| **Database Transformation** | DBT (Data Build Tool) | Implements SQL models to transform staging data to marts in Snowflake. | Data Engineering / Modeling | Snowflake, Azure DevOps Pipelines |
| **Backend Database** | Azure SQL Database | Stores relational application schemas, scenario planner outputs, configuration parameters, and Row-Level Security (RLS) tables. | Storage / Database | JDBC Spark Driver, Azure Key Vault |
| **Middleware API** | Azure Functions / Python | Serves metadata, executes light transactional queries, and initiates simulation actions. | Integration / API | Azure SQL, Snowflake |
| **Application Middleware** | Unifi Platform API | Enforces Policy Enforcement Points (PEP), manages report layouts, and secures requests. | Integration / API | Okta, Azure SQL Database |
| **User Identity** | Okta SSO / Active Directory (AD) | Authenticates users and assigns them to AD groups (`IRIS`, `PPS`, `PEA`). | Security | Cloudflare WAF, Unifi API |
| **Edge Protection** | Cloudflare WAF / CDN / DNS | Provides edge protection, DNS routing, and static asset caching for `iris.app.kimclark.com`. | Security / CDN | React Web App Gateways |
| **Frontend Web App** | React | Renders the interactive user interface for Demand Modeling, Decomposition Insights, Netflow, Price & Promotion Simulator, etc. | Presentation | Unifi Platform API |
| **Business Intelligence** | Power BI Server | Renders model health dashboards for Data Science teams and TPM planners. | Presentation | Snowflake (Direct Query / Import) |
| **Credentials & Secrets** | Azure Key Vault | Stores SAS tokens, Snowflake passwords, and JDBC connection strings securely. | Security | dbutils.secrets |

---

## 2. Component Interdependency Map

1. **Ingestion & Prep**:
   - `ADF` triggers copy tasks -> `Azure Blob Storage Gen2` (Raw) -> `PySpark` transforms raw POS data -> Writes to `Azure Blob Storage Gen2` (Parquet Metrics).
2. **Feature Engineering & Modeling**:
   - `PySpark` reads metrics -> `Prophet (PandasUDF)` runs seasonality -> Writes Seasonality Parquet.
   - `PySpark` joins features + seasonality -> `Bambi/PyMC (PandasUDF)` or `SciPy (PandasUDF)` fits models -> Appends to Model History Parquets.
   - `PySpark` reads history -> Computes Decompositions -> Writes Decomp Parquet.
3. **Synchronization & Warehousing**:
   - `PySpark` reads coefficients/metrics -> Writes to `Snowflake` via Snowflake JDBC -> `DBT` materializes tables.
   - `PySpark` reads coefficients/metrics -> Writes to `Azure SQL Database` via JDBC for Scenario Planner.
4. **Presentation & Serving**:
   - `React UI` queries `Unifi API` -> `Unifi API` checks `Okta AD Group` -> Queries `Azure SQL` (with Row-Level Security) or `Snowflake` -> Returns insights to user.
   - `Power BI` connects directly to `Snowflake` -> Displays model health metrics.
