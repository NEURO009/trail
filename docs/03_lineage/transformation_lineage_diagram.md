# Transformation Lineage Diagram — IRIS Platform

This document describes the transformation equations, logic chains, and statistical adjustments applied to turn raw transaction columns into analytical features and predictions.

---

## 1. Core Feature Transformation Flows

```
[Raw POS Columns]
  ├── AMOUNT, EQ ──────────────────────────► Base Price = AMOUNT / EQ
  │                                            │
  │                                            ▼
  │                                      [stick_smooth (3-week rolling average)]
  │                                            │
  │                                            ▼
  │                                      [log(1 + BASE_PRICE) scaling]
  │                                            │
  │                                            ▼
  │                                      Model Feature: LOG_BASE_PRICE
  │
  ├── UNITS, EQ, ACV_PCT ──────────────────► Promo lift variables:
  │                                            ├── PROMO_PRICE = ANY_PRICE_DECR_AMOUNT / EQ
  │                                            └── PROMO_EQ = EQ * (1 - BASE_EQ_RATIO)
  │
  └── Date (Calendar) ─────────────────────► [COVID Signal Correction (2020-2021)]
                                               │
                                               ▼
                                             [WOM Index Normalization (WoM3 bench)]
                                               │
                                               ▼
                                             [Prophet Yearly + Monthly Fit]
                                               │
                                               ▼
                                             Model Feature: SEASONALITY_INDEX
```

```
[Physical SKU Specs]
  └── Ply, Roll count, sheet size ───────► XGBoost Distance Model 
                                               │
                                               ▼
                                             Pairwise distance scores (0 to 1)
                                               │
                                               ▼
                                             [Similarity Inversion (1 - distance)]
                                               │
                                               ▼
                                             [Interaction weights normalization]
                                               │
                                               ▼
                                             Competitor Weights (sum = 1.0)
                                               │
                                               ▼
                                             Model Feature: COMP_BASE_PRICE_WEIGHTED
```

```
[Model Execution & Attribution]
  ├── Deployed Coefs (Current) 
  └── Wide Feature Matrix ────────────────► Contribution Attribution:
                                               │
                                               ▼
                                             waterfall_value = COEF * FEATURE_DELTA
                                               │
                                               ▼
                                             [Baseline level isolation]
                                               │
                                               ▼
                                             Output: Decomposed Volume Waterfall
```

---

## 2. Key Transformation Formula Reference

### 2.1. Base Price per Equivalent Unit (EQ)
$$\text{Base Price} = \frac{\text{AMOUNT}}{\text{EQ}}$$
- **Stick Smoothing**: Removes temporary weekly spikes by computing a 3-week symmetric window average:
$$\text{Smoothed Price}_{t} = \frac{\text{Price}_{t-1} + \text{Price}_{t} + \text{Price}_{t+1}}{3}$$

### 2.2. Competitor Similarity Inversion
$$\text{Similarity}_{i, j} = 1.0 - \text{Percent Rank}(\text{Euclidean Distance}_{i, j})$$

### 2.3. Competitor Interaction Weight
$$\text{Interaction Weight}_{i, j} = \frac{\text{Similarity}_{i, j} \times \text{Volume Share}_{j}}{\sum_{k \in \text{Competitors}} \text{Similarity}_{i, k} \times \text{Volume Share}_{k}}$$
- Normalizes weights to sum to 1.0 per competitor set.

### 2.4. COVID Anomaly Adjustment
- For periods between `2020-02-01` and `2021-02-01`, seasonality baseline units are adjusted to prevent panic-buying spikes from polluting the Prophet seasonality calculations:
$$\text{COVID Impact Factor} = \frac{\text{Base EQ}_{t}}{\text{Base EQ}_{t-52}}$$
- Units are scaled back using this ratio if it exceeds 1.0.

### 2.5. Attributed Volume Impact (KPI Alerts)
- Used in proactive alerts to estimate the business impact of a KPI deviation:
$$\text{Volume Impact} = \text{Deployed Coefficient} \times (\text{Feature Value}_{t} - \text{Feature Value}_{\text{prior}}) \times \text{Base EQ}_{t}$$
