# Migration Roadmap — IRIS Platform

This document describes the phased migration strategy, deployment roadmap, risk management plans, and verification steps for transitioning the IRIS platform to the target Medallion Architecture.

---

## 1. Phased Migration Plan

To ensure zero business disruption during migration, the project is scheduled across 6 chronological phases:

```
[Phase A: Setup] ──► [Phase B: Bronze] ──► [Phase C: Silver] ──► [Phase D: Gold] ──► [Phase E: Sync] ──► [Phase F: Go-Live]
```

### 1.1. Phase A: Governance & Infrastructure (Month 1)
- Set up Unity Catalog DEV/PROD environments and catalogs.
- Configure AD group security mappings in Unity Catalog.
- Establish cluster profiles and configure Init scripts.

### 1.2. Phase B: Bronze Ingestion Migrations (Month 2)
- Re-route raw files to bronze Delta tables with audit columns.
- Implement schema evolution checks for POS uploads.
- Keep raw Parquet mounts active for parallel run testing.

### 1.3. Phase C: Silver Conformance Migrations (Month 3)
- Port cleaning and deduplication logic into Silver conformed tables.
- Standardize price per EQ metrics and centralize smoothing functions.
- Deploy competitor interaction weights and Prophet seasonality pipelines to Silver.

### 1.4. Phase D: Gold Modeling Migrations (Month 4-5)
- Assemble wide modeling dataframes directly from Silver tables.
- Deploy Bayesian PyMC and Ridge OLS execution scripts in parallel.
- Port decomposition attributions and write results to Gold Delta tables.

### 1.5. Phase E: Scenario Planner & Alerts Sync (Month 6)
- Update scenario planner rollups to read from Gold coefficients.
- Update JDBC export tasks to Snowflake and SQL Server databases.
- Modernize the alerts ThreadPool engine to use distributed Spark UDFs.

### 1.6. Phase F: Parallel Runs & Validation (Month 7)
- Run old and new pipelines in parallel for 4 consecutive weeks.
- Verify coefficient calculations match to within 1e-6 precision.
- Decommission raw Parquet mounts and switch applications to Unity Catalog.

---

## 2. Risk Mitigation & Testing Validation

- **Risk: Model Output Divergence**:
  - *Mitigation*: Run statistical checks comparing old Parquet model runs vs. new Delta model runs on identical test dates. Validate that posterior means match.
- **Risk: Performance Bottlenecks on Prophet/PyMC**:
  - *Mitigation*: Adjust node types during modeling tasks to high-memory instances and monitor execution times.
- **Rollback Strategy**:
  - Keep old ADF schedules active in disabled state. If a production release fails, re-route triggers to old staging mounts.
