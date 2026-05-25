# 04 — Advanced Analytics (Alchemists — Global Data Team)

## Overview

In the second phase of my time at Rappi, I joined the **Alchemists** — the Global Data Team for the Restaurant vertical. This team operated at a different scope: instead of maintaining operational pipelines, the focus was on **strategic insights, predictive models, and analytical frameworks** that informed decisions at the global level across all 9 countries.

This section documents the four main analytical workstreams I contributed to.

---

## 1 — MLR Model: Impact of Partner Gains/Losses on Orders

### Business Question
> *What would be the increase (or decrease) in orders in the event of gaining or losing a restaurant partner in a given microzone?*

This was one of the most strategically important questions the team worked on — with direct implications for the Hunting team's prioritization of which new partners to onboard first, and for understanding churn risk at the market level.

### Approach — Multiple Iterations

**Version 1: Multiple Linear Regression (MLR)**
The initial model used standard MLR to predict order volume as a function of partner density, cuisine type coverage, and microzone characteristics.

```python
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score

# Features: partner count, cuisine coverage, microzone demand indicators
X = df[['partner_count', 'cuisine_coverage', 'microzone_demand', 'population_proxy']]
y = df['orders_delta']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)

r2 = r2_score(y_test, model.predict(X_test))
# R² ranged 0.70–0.82 depending on market cluster
```

**Version 2: MLR with Interaction Terms**
Added interaction terms to capture non-linear relationships — e.g., the effect of adding a partner in a saturated microzone vs. an underserved one differs significantly.

**Version 3: Polynomial and Poisson Regression**
Explored polynomial regression (for non-linear demand curves) and Poisson regression (appropriate for count data like orders). Both were evaluated but ultimately the MLR with interaction terms provided the best interpretability-accuracy tradeoff.

**Final approach: Missing Cuisine Model (Assortment Model)**
The team ultimately pivoted to a recommendation-algorithm-based approach — the **Missing Cuisine Model** — which identified microzones with suboptimal cuisine assortment and predicted the conversion increase from adding the missing cuisine type.

```sql
-- Snowflake query to extract model outputs by microzone
SELECT
    microzone_id,
    cluster_name,
    increase_conversion
FROM MX_WRITABLE.CLUSTERING_SUBOPTIMAL_ASSORTMENT
WHERE prediction_date = CURRENT_DATE
  AND country = 'CO'
ORDER BY increase_conversion DESC;
```

**Results:**
- R² = 0.70–0.82 across 4 market clusters
- Significant variation in effect size across clusters — a partner gain in a high-demand/low-supply microzone had 3–4x the order impact compared to a saturated microzone
- Model outputs used by Hunting team for microzone prioritization

---

## 2 — CUSUM Churn Detection Model

### Business Question
> *Can we detect early signals of store churn (order decline) before a store goes inactive — giving the team time to intervene?*

### Approach
The **CUSUM (Cumulative Sum) model** is a sequential change-detection algorithm that identifies when a metric has shifted from its baseline — ideal for detecting gradual order declines before they become visible in standard weekly reports.

**Dataset built for the model:**

```sql
-- DATASET_CHURN.sql — fields extracted from Snowflake
SELECT
    store_id,
    country,
    week,
    orders,           -- Weekly order volume
    ss,               -- Same-store metric
    cvr,              -- Conversion rate
    late_arrivals     -- Late delivery rate (quality signal)
FROM analytics_layer.store_weekly_performance
WHERE active = TRUE
ORDER BY store_id, week;
```

**Model pipeline (Python):**

```python
# version_flg_CM_MCV1.ipynb
# 1. Define baseline per store (first N weeks post-activation)
# 2. Calculate cumulative sum of deviations from baseline
# 3. Flag when CUSUM exceeds threshold → churn signal
# 4. Evaluate with confusion matrix

from sklearn.metrics import confusion_matrix, classification_report

# Confusion matrix results documented in model notebook
# Box plots used to visualize CUSUM threshold calibration per cluster
```

**Key outputs:**
- Confusion matrix evaluated at multiple threshold values
- Box plots per market cluster showing CUSUM distribution
- Binary flag (`flg_CM_MCV1`) appended to store records for downstream use
- Early signal lead time: model flagged at-risk stores several weeks before standard weekly review would have caught the decline

---

## 3 — Market Clustering

### Business Question
> *Are all markets (microzones/countries) the same, or should analytical models and benchmarks be segmented?*

Applying a single model or benchmark across 9 countries and hundreds of microzones masked critical differences in market maturity, density, and demand patterns.

**Approach:**
4-cluster segmentation of markets using order volume, conversion rates, partner density, and cuisine coverage as features. Each cluster received its own MLR model coefficients and churn thresholds.

**Cluster profiles (generalized):**
| Cluster | Profile | Implication |
|---------|---------|-------------|
| Cluster A | High demand, high supply | Marginal value of adding partners is low; focus on quality over quantity |
| Cluster B | High demand, low supply | Highest marginal value — priority for Hunting |
| Cluster C | Low demand, established | Stable base; churn risk moderate; intervention ROI lower |
| Cluster D | Emerging markets | High growth potential; different activation playbook needed |

---

## 4 — WBR Metrics & Global Reporting

### Business Question
> *What are the key metrics the Restaurant vertical should report weekly to global leadership (WBR — Weekly Business Review)?*

### Deliverables

**Metric Documentation Standard**
Consolidated and formally defined all metrics assigned to the Alchemists team in Q4. Each metric documented with: definition, SQL source, ownership, update cadence, and known edge cases.

**WBR Query**
A large, multi-CTE Snowflake query extracting all WBR metrics in a single run — covering orders, conversion rates, partner performance, churn signals, and market health indicators across all countries.

```sql
-- WBR Query structure (simplified — 200KB+ in production)
WITH
base_orders AS (
    SELECT store_id, country, week, orders, gmv
    FROM GLOBAL_FINANCES.GLOBAL_ORDERS
    WHERE ...
),
base_performance AS (
    SELECT store_id, full_perf_final, ...
    FROM RESTAURANTES_GLOBAL_MDA.STORE_LEVEL_FULL_PERF
),
base_conversion AS (
    SELECT store_id, cvr, ...
    FROM CPGS_CONVERSIONS.TBL_RAPPI_BASE_CONVERSIONS_LEVEL_ALL
),
-- ... 10+ additional CTEs
final AS (
    SELECT
        o.country,
        o.week,
        SUM(o.orders)           AS total_orders,
        AVG(c.cvr)              AS avg_cvr,
        COUNT(p.store_id)       AS active_partners,
        ...
    FROM base_orders o
    LEFT JOIN base_performance p USING (store_id)
    LEFT JOIN base_conversion c USING (store_id)
    GROUP BY 1, 2
)
SELECT * FROM final ORDER BY country, week DESC;
```

**Pacing Assortment Metrics**
Separate query tracking cuisine assortment completeness by microzone — feeding the Missing Cuisine Model and the Hunting prioritization framework.

**Funnel Hunting Query**
Extracted the full Hunting funnel metrics (signings → activations → first sale) from `RESTAURANTES_GLOBAL_MDA.CONTROL_TOWER_HUNTING` for global leadership reporting.

**Table Cleanup Initiative**
Identified and documented fields recommended for removal from `RFS_RTS_STORES` and `RFS_AVAILABILITY_30D` — a data hygiene initiative to reduce query complexity and storage costs across the analytical layer.

---

## Key Analytical Principles Applied

**1. Segment before modeling.** Applying global models to all 9 markets simultaneously masked the most actionable insights. Country and cluster-level segmentation was non-negotiable.

**2. Interpretability over accuracy.** For strategic decisions (Hunting prioritization, churn intervention), a model a business stakeholder can understand and challenge is more valuable than a black-box model with marginally better R².

**3. Document the dead ends.** The CUSUM model notebook and the MLR iteration documents captured not just what worked — but which approaches were tried and abandoned, and why. That documentation was as valuable as the final model.

**4. Connect analytics to action.** Every model and metric had a defined downstream use: CUSUM flags → Customer Success intervention. MLR outputs → Hunting microzone ranking. WBR queries → Global leadership reporting. Analytics without a clear action path is just data gymnastics.

