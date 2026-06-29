# Architecture Decision Records (ADR) Index — IRIS Platform

This directory contains records of structural architectural decisions made during the IRIS Platform Modernization.

---

## 1. ADR Index

- [ADR 001: Medallion Architecture with Delta Lake](file:///Users/navneetsharma/Coder/trail/docs/08_adr/adr_001_medallion_architecture.md)
  - **Status**: Approved.
  - **Context**: Replaces unstructured Parquet mount folders with structured, governed Delta layers (Bronze, Silver, Gold).
- **ADR 002: Databricks Native Workflows Orchestration** (future draft)
  - **Status**: Proposed.
  - **Context**: Replaces ADF + nested `dbutils.notebook.run` calls with Databricks Workflows DAG.
- **ADR 003: Unity Catalog Governance & Row-Level Filtering** (future draft)
  - **Status**: Proposed.
  - **Context**: Enforces schema RBAC and sensitive market constraints directly inside the metastore layer.
- **ADR 004: Databricks Asset Bundles for CI/CD Deployment** (future draft)
  - **Status**: Proposed.
  - **Context**: Standardizes deployment using DAB and Azure DevOps pipelines.
