# Delta Lake Structure & Optimization — IRIS Platform

This document defines the storage patterns, optimization policies, and data management configurations for the target Delta Lake implementation.

---

## 1. Delta Lake Design Principles

By converting raw `.parquet` folders to managed `.delta` tables, we enforce:
1. **ACID Transactions**: Guarantees concurrent write safety.
2. **Schema Enforcement**: Prevents corrupt data writes.
3. **Time Travel**: Retains past versions (30 days default) for debugging and audit runs.

---

## 2. Optimization and Maintenance Operations

Large retail scan tables require scheduled optimization scripts running weekly:

### 2.1. Liquid Clustering
- **Action**: Replace traditional partitioning (`partitionedBy("category_id")`) with Delta Liquid Clustering on high-cardinality query keys:
```sql
ALTER TABLE gold.deployed_coefficients 
CLUSTER BY (market_key, product_key);
```
- **Benefit**: Speeds up coordinate reads during scenario simulation lookups and alert checks.

### 2.2. Optimize & Vacuum
- **OPTIMIZE**: Compacts small files into large sequential blocks. Run weekly on active transaction tables.
```sql
OPTIMIZE silver.conformed_nielsen_metrics;
```
- **VACUUM**: Clears stale transactional logs and physical files older than the retention period (7 days default) to recover storage space.
```sql
VACUUM gold.deployed_coefficients RETAIN 168 HOURS;
```

---

## 3. Schema Governance Policy

- **Schema Enforcement**: Enabled by default to block writes containing unexpected fields or mutated types.
- **Schema Evolution**: Allowed only via explicit option for intentional additions:
```python
df.write.format("delta") \
  .mode("append") \
  .option("mergeSchema", "true") \
  .saveAsTable("silver.conformed_nielsen_metrics")
```
- **Data Constraints**: Enforces key invariants:
```sql
ALTER TABLE silver.conformed_nielsen_metrics 
ADD CONSTRAINT positive_volume CHECK (eq_volume >= 0);
```
