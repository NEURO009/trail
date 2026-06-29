# Databricks Workspace Structure — IRIS Platform

This document describes the design configurations for the target Databricks workspace, including cluster profiles, repository integration, permissions, and environment segregation.

---

## 1. Environment Segregation (Dev vs. Prod)

To prevent data contamination and ensure compliance in the secure Sensitive zone:
1. **Development Workspace**:
   - Connected to the non-production Azure subscription VNET.
   - Computes runs on smaller, auto-scaling clusters.
   - Connected to DEV databases and dev storage mounts.
2. **Production Workspace**:
   - Hosted in the secure production subscription.
   - Compute nodes run on designated high-concurrency clusters.
   - Connected strictly to production catalogs and databases.

---

## 2. Cluster Profiles & Compute Guidelines

To support statistical packages (PyMC, Bambi) and scale Prophet modeling:
- **Shared Job Clusters**:
  - Allocated dynamically by Databricks Workflows per pipeline job run to save costs.
  - Runtime: Databricks Runtime 14.3 LTS ML (contains scikit-learn, XGBoost, and Prophet pre-installed).
  - Initialization Scripts (Init Scripts): Executes custom installations for `pymc`, `bambi`, `arviz`, `xarray`.
  - Node Configuration: Memory-Optimized VM types (e.g. `Standard_E8ds_v5` on Azure) to prevent out-of-memory errors during MCMC sampling.
- **Single Node Clusters**:
  - Reserved for development workflows and simple data discovery tasks.

---

## 3. Git Integration (Databricks Repos / Git Folders)

- Databricks Repos (Git Folders) are mapped to user directories (`/Workspace/Repos/user@kimclark.com/iris-analytics/`).
- Master branches are locked and updated strictly via automated CI/CD PR merges.
- Databricks Asset Bundles (DAB) utilize standard git checkouts to push production runs.
