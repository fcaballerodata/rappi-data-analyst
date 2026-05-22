# Rappi — Data Business Analyst
### Restaurant Vertical · Onboarding Team → Global Data Team (Alchemists)
**Apr 2022 – Mar 2024 · Remote · 9 countries**

> ⚠️ **Anonymized Case Study**  
> All internal table names, queries, and references in this repository have been anonymized or generalized. This case study is shared exclusively to demonstrate technical skills and methodology. It does not disclose any confidential or proprietary information belonging to Rappi.

---

## Context

Rappi is one of Latin America's leading super-apps, operating across 9 countries with millions of daily active users. The Restaurant vertical alone manages hundreds of thousands of partner stores across the region.

I joined as a **Data Business Analyst** on the **Onboarding Team** — the squad responsible for activating new restaurant partners from signing to first sale. Over 2 years, my role evolved from maintaining and building the analytical data layer in Snowflake, to owning dashboards used daily by BPO teams and senior leadership, to running statistical experiments, and finally joining the **Global Data Team (Alchemists)** where I worked on advanced analytics and strategic insights for the full restaurant vertical.

---

## Two Years — Two Phases

```
Apr 2022 ──────────────────────────── ~2023 ──────────────── Mar 2024
│                                        │                        │
│    PHASE 1 — ONBOARDING TEAM           │   PHASE 2 — ALCHEMISTS │
│                                        │                        │
│  · 15+ Snowflake scheduled tasks       │  · MLR models          │
│  · 5+ Power BI dashboards              │  · CUSUM churn model   │
│  · A/B experiment with BPO Invisible   │  · WBR global metrics  │
│  · Data Dictionary (team standard)     │  · Clustering analysis │
└────────────────────────────────────────┴────────────────────────┘
```

---

## What Was Built

| Area | Deliverable |
|------|-------------|
| ❄️ [Snowflake Data Layer](./01-snowflake-data-layer/) | 15+ scheduled tasks (DDL/SQL) · Analytical layer for Ready for Success vertical · Multi-source integration |
| 📊 [Dashboards](./02-dashboards/) | 5+ Power BI Control Towers: Ready for Success, QA Readiness, SOB, SLA R2S, Early Success/Invisible |
| 🧪 [A/B Experiment](./03-ab-experiment/) | Activation experiment with BPO Invisible · t-test + z-test at 95% confidence · Organic vs. treatment stores |
| 📈 [Advanced Analytics](./04-advanced-analytics/) | MLR models (R²=0.70–0.82) · CUSUM churn detection · Market clustering · WBR global metrics |

---

## Impact

- **15+ Snowflake scheduled tasks** designed, built, and maintained in production across the analytical layer of the Restaurant Onboarding vertical
- **5+ Power BI dashboards** consumed daily by BPO teams, operations leads, and regional directors across 9 countries
- **A/B experiment** designed and executed with BPO Invisible — results directly informed the organic store activation strategy
- **Data Dictionary** created and adopted as the team standard for the Onboarding vertical
- **MLR models** with R²=0.70–0.82 across 4 market clusters, quantifying the revenue impact of partner gains/losses
- **Recognition:** Virtues Master Q4 2022 · Golden Moustache Q3 2022

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Data Warehouse | Snowflake (DDL, scheduled tasks, task trees, pipeline monitoring) |
| BI & Visualization | Power BI (DAX, Power Query, scheduled refresh) |
| CRM Integration | ZOHO CRM (Accounts, Deals, Onboarding, QA Readiness, Contacts) |
| Analytics & Stats | Python (Pandas, NumPy, Scikit-learn, Matplotlib) · t-test · z-test · MLR |
| Other Sources | Microservices (public store APIs) · Google Sheets · BPO contactability data |

---

## Repository Structure

```
rappi-data-analyst/
│
├── 01-snowflake-data-layer/
│   └── README.md     ← 15+ Snowflake tasks: DDL, sources, scheduling, architecture
│
├── 02-dashboards/
│   └── README.md     ← 5 dashboards: purpose · tabs · KPIs · audience · refresh
│
├── 03-ab-experiment/
│   └── README.md     ← Experiment design · hypothesis · statistical test · results
│
└── 04-advanced-analytics/
    └── README.md     ← MLR · CUSUM churn · market clustering · WBR metrics
```

---

## Connect

**Fredys Caballero** · Business Intelligence Analyst · Data Analyst  
[LinkedIn](https://linkedin.com/in/fcaballerosoto) · [GitHub Portfolio](https://github.com/fcaballerodata) · fredyscaballero@gmail.com
