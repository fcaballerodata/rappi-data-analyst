# 03 — A/B Experiment: Organic vs. Treatment Store Activation

## Overview

One of the most analytically rigorous deliverables of my time at Rappi was designing and executing a **controlled A/B experiment** in collaboration with the Onboarding team and BPO Invisible — testing whether active outreach (treatment) meaningfully increased first-sale activation rates compared to stores that activated organically (control).

The results directly informed the organic store activation strategy for the vertical.

---

## Business Question

> **Does BPO Invisible's active outreach to newly onboarded stores significantly increase their probability of making a first sale, compared to stores that activate without contact?**

This question had major operational implications: BPO Invisible represented a significant cost. If organic activation rates were not meaningfully lower than treatment rates, the team could reallocate resources. If treatment significantly outperformed organic, the experiment would quantify exactly how much.

---

## Experiment Design

| Element | Definition |
|---------|-----------|
| **Unit of analysis** | Individual restaurant store |
| **Control group** | Stores that completed onboarding and received no BPO outreach (organic activation) |
| **Treatment group** | Stores that completed onboarding and received BPO Invisible outreach (calls/messages) |
| **Primary metric** | First-sale activation rate within a defined time window |
| **Secondary metrics** | Time-to-first-sale · Contactability rate · Conversion per BPO agent |
| **Confidence level** | 95% (α = 0.05) |
| **Statistical tests** | t-test (means comparison) · z-test (proportions comparison) |

---

## Data Infrastructure for the Experiment

The experiment required building a dedicated data pipeline to track treatment assignment and outcomes at the store level:

**Source: `RAPPI_AS_CONTACTABILITY`**
- Raw contactability data from BPO Invisible (Google Sheets → Snowflake)
- Fields: store ID · contact date · contact attempt result · agent ID · channel (Hunting/SOB)
- Backup strategy: when Google Sheet hits row limit, backup table created and appended via Power Query

**Tracking dashboard: Early Success — Invisible**
- "Overview Contactability" and "Contactability — Per Country" tabs tracked experiment progress in real time
- "Agents Performance" tab monitored treatment delivery fidelity (were stores actually being contacted?)

---

## Hypothesis

```
H₀ (Null):       Activation rate (Treatment) = Activation rate (Control)
H₁ (Alternative): Activation rate (Treatment) > Activation rate (Control)

Test type: One-tailed (we expected treatment to outperform)
α = 0.05 → z-critical = 1.645 (proportions test)
```

---

## Statistical Tests Applied

### z-test — Proportions Comparison
Used to compare the **activation rate** (binary outcome: first sale yes/no) between control and treatment groups.

```python
from scipy import stats
import numpy as np

# Proportions
p_control   = activated_control / n_control
p_treatment = activated_treatment / n_treatment
p_pooled    = (activated_control + activated_treatment) / (n_control + n_treatment)

# Standard error
se = np.sqrt(p_pooled * (1 - p_pooled) * (1/n_control + 1/n_treatment))

# z-statistic
z_stat = (p_treatment - p_control) / se

# p-value (one-tailed)
p_value = 1 - stats.norm.cdf(z_stat)
```

### t-test — Time-to-First-Sale Comparison
Used to compare the **average time** (in days) from credential delivery to first sale between groups.

```python
from scipy.stats import ttest_ind

t_stat, p_value = ttest_ind(
    time_to_first_sale_treatment,
    time_to_first_sale_control,
    alternative='less'   # Treatment expected to have shorter time
)
```

---

## Results

> Note: Specific numerical results are not disclosed to protect internal business data. The methodology and approach are documented here.

**Key findings from the experiment:**
- The z-test produced a statistically significant result at the 95% confidence level — the null hypothesis was rejected
- Treatment stores showed a meaningfully higher activation rate than organic stores within the measurement window
- Time-to-first-sale was shorter in the treatment group, consistent with the hypothesis that active outreach reduces friction in the activation process
- Results were segmented by country and channel (Hunting vs. SOB) — effect size varied across markets

**Business impact:**
The experiment results were presented to the Onboarding team leadership and directly informed the resource allocation decision for BPO Invisible's activation outreach program. The statistical evidence provided the justification needed to maintain and scale the BPO engagement model rather than relying on organic activation.

---

## Lessons from This Experiment

**1. Experiment design is harder than the statistics.**
The most difficult part was defining the control group cleanly. Stores that received no contact had to be verified as truly "not contacted" — not just not in the BPO list. Data quality in `RAPPI_AS_CONTACTABILITY` was the critical dependency.

**2. Fidelity matters.**
The "Agents Performance" dashboard tab existed precisely for this reason: to monitor whether treatment was actually being delivered consistently. If some agents had low contact rates, the treatment group was contaminated.

**3. Segment before concluding.**
Aggregate results masked important market-level differences. A country-level breakdown revealed that effect sizes varied significantly, which had implications for how resources should be allocated across markets.
