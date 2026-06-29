# Unity Catalog Governance Structure — IRIS Platform

This document describes the governance design, access control groups, and row-level security structures implemented using Databricks Unity Catalog.

---

## 1. Catalog Hierarchy Namespace

Unity Catalog structures data using a three-level namespace: `catalog.schema.table`.

```
                  ┌───────────────────────────────────┐
                  │          METASTORE (Azure)        │
                  └─────────────────┬─────────────────┘
                                    │
          ┌─────────────────────────┴─────────────────────────┐
          ▼                                                   ▼
┌───────────────────┐                               ┌───────────────────┐
│ CATALOG: iris_dev │                               │ CATALOG: iris_prod│
└─────────┬─────────┘                               └─────────┬─────────┘
          │                                                   │
   ┌──────┼──────┐                                     ┌──────┼──────┐
   ▼      ▼      ▼                                     ▼      ▼      ▼
┌──────┬──────┬──────┐                              ┌──────┬──────┬──────┐
│bronze│silver│ gold │                              │bronze│silver│ gold │
└──────┴──────┴──────┘                              └──────┴──────┴──────┘
```

- **Catalogs**:
  - `iris_dev`: Sandbox catalog for development testing.
  - `iris_prod`: Production catalog with production data.
- **Schemas**:
  - `bronze`: Stores raw tables.
  - `silver`: Stores conformed and cleaned datasets.
  - `gold`: Stores aggregated business-ready reporting tables.
  - `metadata`: Contains configs and logging directories.

---

## 2. Security Roles & Access Control Lists (ACLs)

Permissions are defined and enforced at the catalog level:

| User Group / Role | Allowed Catalogs | Permissions |
|---|---|---|
| **Data Engineers** | `iris_dev`, `iris_prod` | `ALL PRIVILEGES` (Manage tables, schemas, and pipelines) |
| **Data Scientists** | `iris_dev`<br>`iris_prod` | `ALL PRIVILEGES` on DEV.<br>`SELECT` on PROD Silver/Gold (Read-only access for analytics modeling). |
| **Application Principals** | `iris_prod` | `SELECT` on Gold tables (Read-only access for Unifi API integrations). |
| **Business Planners** | `iris_prod` | `SELECT` on Gold tables (Read-only access for Power BI reports). |

---

## 3. Row-Level Security & Column-Level Masking

To prevent users in the Sensitive landscape from accessing unlicensed market/product categories:
- **Row-Level Security (RLS)**: Enforces row filters based on user AD group mappings.
- **Implementation**:
```sql
CREATE FUNCTION gold.market_category_filter(market_val VARCHAR, category_val VARCHAR)
RETURN IS_ACCOUNT_GROUP_MEMBER('admin') OR
       EXISTS (
         SELECT 1 FROM metadata.user_security_mapping 
         WHERE user_email = CURRENT_USER() 
           AND (market_key = market_val OR market_key = 'ALL')
           AND (category_id = category_val OR category_id = 'ALL')
       );

ALTER TABLE gold.deployed_coefficients 
SET ROW FILTER gold.market_category_filter ON (market_key, category_id);
```
- **Column-Level Masking**: Mask personal subscriber emails (`security.USERS`) for general developer roles.
