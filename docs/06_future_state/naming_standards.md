# Naming Standards & Conventions — IRIS Platform

This document establishes coding style rules, database table namespaces, and naming conventions to ensure consistency across development teams.

---

## 1. Directory and Repository Namespaces

All repositories and artifacts must follow clean lowercase namespaces:
- **Git Repositories**: `iris-analytics-platform`, `iris-dbt-snowflake`.
- **Databricks Workspaces**: `/Workspace/Repos/production/iris-analytics-platform/`.
- **Azure Blob Mounts**: `/mnt/iris-{source}-models/`.

---

## 2. Table Namespaces in Medallion Layer

Tables are written in lowercase, using singular nouns, and utilizing descriptive suffixes:
- **Bronze Layer**: `bronze.{source}_{table_name}`
  - Example: `bronze.nielsen_pos_syndicated`, `bronze.amazon_pos`.
- **Silver Layer**: `silver.conformed_{metrics_name}` or `silver.{dimension_name}`
  - Example: `silver.conformed_nielsen_metrics`, `silver.product_hierarchy`.
- **Gold Layer**: `gold.{business_use_case_name}`
  - Example: `gold.deployed_coefficients`, `gold.model_decompositions`, `gold.kpi_alerts`.

---

## 3. Column and Variable Conventions

- **SQL Columns**: UPPER_SNAKE_CASE (for database column compatibility).
  - Key IDs: `MARKET_KEY`, `PRODUCT_KEY`, `WEEK_ID`.
  - Aggregates: `POS_AMOUNT_SMOOTH`, `POS_EQ_VOLUME`.
- **Python Variables**: lower_snake_case (adhering to PEP 8 standards).
  - Dataframes: `base_metrics_df`, `model_features_df`.
  - Parameters: `category_id`, `source_key`.
- **Delta Constraint Mappings**:
  - Primary Key markers: `pk_{table_name}`.
  - Foreign Key checks: `fk_{table_name}_{referenced_table}`.
