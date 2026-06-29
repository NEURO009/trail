# IRIS Analytics Platform Modernization — Enterprise Architecture Documentation

This document serves as the master Enterprise Architecture reference for the modernized IRIS Analytics Platform.

---

## 1. Document Control & Approvals

- **Project Title**: IRIS Platform Modernization & Medallion Architecture
- **Version**: 1.0.0
- **Classification**: Enterprise Restricted
- **Author**: Principal Data Architect
- **Approved by**: Chief Data Officer, Enterprise Architecture Board

---

## 2. Platform Architecture Summary

The IRIS Analytics Platform is Kimberly-Clark's enterprise-scale analytics system for retail sales modeling, pricing optimization, and trade promotion analysis. It processes Nielsen, Kroger, Amazon, Profitero, and Circana POS datasets to train Bayesian (PyMC) and Constrained OLS models, delivering insights via scenario planners, proactive alerts, and BI reporting dashboards.

The platform is structured into two environments:
1. **Internal Zone**: Standard data pipelines, aggregations, and general Snowflake marts.
2. **Sensitive Zone**: Isolated Zero-Trust subscription for proprietary trade runs. Protected by Cloudflare WAF, authenticated via Okta SSO, and secured using row-level filters matching user privileges.

---

## 3. Current-State Gaps & Technical Debt

A detailed assessment of the current state identified:
- **No Schema Governance**: Raw Parquet formats lack validation. Mutation of raw data structures causes silent failures.
- **Logic Duplication**: Price calculations, smoothing UDFs, and deduplication logic are repeated across feature engineering, modeling wide dataframe assembly, and proactive alerts processing modules.
- **Fragile Orchestration**: Scheduling relies on ADF executing nested `dbutils.notebook.run` calls, which lacks robust failover.
- **Access Control Gaps**: Storage mount authorization relies on SAS tokens, bypassing table-level access controls.

---

## 4. Target Medallion Design

The modernization reorganizes the storage and compute layer into Delta Lake:
- **Bronze (Raw Ingestion)**: Immutable append-only tables storing raw POS/e-commerce feeds with added landing metadata.
- **Silver (Conformed & Cleansed)**: Deduplicated, casted, and validated features. Centralizes stick price smoothing and competitor mapping weights.
- **Gold (Business Ready)**: Model features wide dataframes, deployed coefficients, and alerting outputs optimized using Liquid Clustering.
- **Metastore Governance**: Enabled via Unity Catalog to enforce catalog permissions and row-level security on client lookups.
- **DAG Orchestration**: Migrated to native Databricks Workflows.
- **Automated CI/CD**: Run using Databricks Asset Bundles (DAB) and Azure DevOps pipelines.

---

## 5. Implementation Roadmap

The modernization follows a structured 7-month roadmap:
1. **Month 1 (Governance)**: Deploy Unity Catalog and configure metastore privileges.
2. **Month 2 (Bronze Ingest)**: Connect ingestion to Bronze Delta tables.
3. **Month 3 (Silver Features)**: Deploy Conformed Silver metrics and Prophet seasonality runs.
4. **Month 4-5 (Gold Modeling)**: Transition Bayesian/OLS models and attributions to Gold Delta.
5. **Month 6 (Integration)**: Sync Gold tables to Snowflake and upload scenario facts to SQL Server.
6. **Month 7 (Go-Live)**: Execute 4 weeks of parallel validation testing before decommissioning old mounts.
