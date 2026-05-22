# 01 — Snowflake Data Layer

## Overview

The analytical layer of the Restaurant Onboarding vertical lived in Snowflake. My role was to **design, build, and maintain 15+ scheduled tasks (DDL/SQL)** that kept the data pipeline running 24/7 for dashboards consumed by BPO teams, operations leads, and regional directors across 9 countries.

This section documents the architecture, the task ecosystem, the data sources integrated, and the engineering practices applied.

---

## A Note on Learning — Standing on the Shoulders of Mentors

Before documenting what I built, I want to acknowledge two people without whom none of this would have been possible.

**César Morales** and **Luis Zavala** — both Senior Data Analysts on the team — were the architects behind `RFS_AVAILABILITY_30D`, the most critical and complex table in the entire analytical layer. This table is the foundation that feeds almost every dashboard and downstream task in the vertical.

What made the difference wasn't just their technical knowledge — it was their genuine disposition to teach. They took the time to explain not just *what* the table did, but *why* it was designed the way it was: the source decisions, the transformation logic, the edge cases they had learned to handle. That foundation is what allowed me to maintain `RFS_AVAILABILITY_30D` with confidence, and ultimately build 15+ additional tasks following the same patterns and best practices they established.

> *César, Luis — thank you. This work reflects your standards.*

---

## The Analytical Layer — Task Ecosystem

The Snowflake analytical layer was built as a **tree of scheduled tasks** — each task running on a defined schedule, feeding downstream tables consumed by dashboards or other tasks.

```
                    ┌─────────────────────────────┐
                    │   ZOHO CRM (9 tables)        │
                    │   Public Microservices (CMS) │
                    │   Google Sheets              │
                    │   BPO Contactability data    │
                    │   Hunting / Signings tables  │
                    └────────────────┬────────────-┘
                                     │
                    ┌────────────────▼─────────────┐
                    │       CORE LAYER              │
                    │                              │
                    │  RFS_RTS_STORES              │ ← Store master (CMS + ZOHO)
                    │  RFS_AVAILABILITY_30D        │ ← Main analytical table *
                    │  RFS_STORES_SUSPENSIONS      │ ← Suspension history
                    │  RFS_CONFIGURED_HOURS_DAILY  │ ← Schedule availability
                    │  RAPPI_AS_CONTACTABILITY     │ ← BPO contact data
                    └────────────────┬─────────────┘
                                     │
                    ┌────────────────▼─────────────┐
                    │     DERIVED LAYER             │
                    │                              │
                    │  RFS_PRODUCT_PERCENTAJES     │ ← Product completion %
                    │  RFS_PRODUCT_STOCKOUTS       │ ← Stockout tracking
                    │  RFS_CANCELLED_ORDERS_F12W   │ ← 12-week order cancellations
                    │  QAR_HT / QAR_SOB / QAR_IR  │ ← QA Readiness by channel
                    │  RFS_ONB_SLA                 │ ← Onboarding SLA times
                    │  RFS_CREDENTIALS_*           │ ← Credential delivery tracking
                    └────────────────┬─────────────┘
                                     │
                    ┌────────────────▼─────────────┐
                    │      SOB FUNNEL LAYER         │
                    │                              │
                    │  RFS_SOB_FUNNEL_DAILY        │
                    │  RFS_SOB_FUNNEL_WEEKLY       │ ← Multi-granularity funnel
                    │  RFS_SOB_FUNNEL_MONTHLY      │
                    │  RFS_SOB_COHORTS_ACTIVATION  │ ← W1/W2/W3/W4 cohorts
                    │  RFS_SOB_BASE_LEADS          │ ← Lead base for SOB
                    │  RFS_SOB_ZOHO_LEADS_PER_HOUR │ ← Lead velocity tracker
                    └──────────────────────────────┘

* RFS_AVAILABILITY_30D: architected by César Morales & Luis Zavala
```

---

## Key Tables Documented

### `RFS_RTS_STORES` — Store Master Table
The foundation of the entire vertical. Integrates store data from ZOHO CRM and public microservices (CMS) to produce a unified store-level record.

**Sources integrated:**
- `ZOHO_CRM.ACCOUNT` · `ZOHO_CRM.DEAL` · `ZOHO_CRM.LEAD_STORES` · `ZOHO_CRM.ONBOARDING`
- `ZOHO_CRM.ACTIVACIONES_VW` · `ZOHO_CRM.FOTOGRAFIA` · `ZOHO_CRM.QA_READINESS`
- `XX_PGLR_MS_STORES_PUBLIC.STORES_VW` · `XX_PGLR_MS_STORES_PUBLIC.STORE_BRANDS_VW`
- `GLOBAL_VALUE_PROP_DS.STORES_CELL` (Microzone mapping)
- `RESTAURANTS_HUNTING.*` (Hunting + Signings)
- `RESTAURANTS_GLOBAL_POSTSALES.A_SCORECARD_REST4` (Performance scoring)

---

### `RFS_AVAILABILITY_30D` — Main Analytical Table ⭐
*Architecture by César Morales & Luis Zavala.*

The most consumed table in the vertical. Provides a 30-day rolling availability view for every active store — combining readiness indicators, schedule compliance, product health, cancellation rates, and conversion metrics.

**Sources integrated:**
- `RFS_RTS_STORES` (store master)
- `RFS_STORES_SUSPENSIONS` (suspension history)
- `CO_WRITABLE.RFS_CONFIGURED_HOURS_DAILY` (schedule availability)
- `RFS_PRODUCT_PERCENTAJES` (product completion)
- `RFS_CANCELLED_ORDERS_F12W` (cancellation rates)
- `GLOBAL_VALUE_PROP_DS.AVAILABLE_STORES_DATASET` (availability dataset)
- `CPGS_CONVERSIONS.TBL_RAPPI_BASE_CONVERSIONS_LEVEL_ALL` (conversion data)
- `GLOBAL_PAYMENTS.GP_FRA_ALDS_RAW_FRAUDSTORES` (fraud exclusions)
- `RESTAURANTES_GLOBAL_MDA.STORE_LEVEL_FULL_PERF` (performance segmentation: TOP field)

**Why this table is the hardest to maintain:**
It consolidates 10+ sources with different owners, update cadences, and schemas. Any change upstream — a renamed field in ZOHO, a schema change in CMS microservices, a new fraud rule — can break downstream tasks silently. Maintaining it required understanding every dependency and proactively monitoring pipeline health.

---

### `RAPPI_AS_CONTACTABILITY` — BPO Contact Data
Feeds the BPO Invisible contactability tracking for Hunting and SOB channels.

**Source:** Google Sheets (raw data from BPO Invisible, refreshed continuously)  
**Architecture note:** When the Google Sheet reaches its row limit, a backup table is created (`RAPPI_AS_CONTACT_BACKUP_1`, `_2`, etc.) and appended via Power Query in the dashboard. This was documented in the handoff guide for the successor.

---

### SOB Funnel Layer — Three Granularities
The Self-Onboarding (SOB) funnel was tracked at three time resolutions simultaneously:

| Table | Granularity | Purpose |
|-------|-------------|---------|
| `RFS_SOB_FUNNEL_DAILY` | Daily | Operational monitoring, anomaly detection |
| `RFS_SOB_FUNNEL_WEEKLY` | Weekly | WTD tracking, team reviews |
| `RFS_SOB_FUNNEL_MONTHLY` | Monthly | MTD reporting, leadership dashboards |

All three fed the SOB Control Tower dashboard and were refreshed on staggered schedules to avoid resource contention.

---

### QA Readiness Tables — Three Channels
| Table | Channel | Primary Users |
|-------|---------|--------------|
| `QAR_HT` | Hunting | Hunters, Brand Expansion |
| `QAR_SOB` | Self-Onboarding | SOB operations team |
| `QAR_IR` | Initial Readiness | Onboarding coordinators |

Each table combined ZOHO CRM data with RFS availability metrics to produce a readiness score used for credential delivery decisions.

---

### `RFS_ONB_SLA` — Onboarding SLA Tracker
Tracks the time spent at each stage of the onboarding funnel — from signing to credential delivery to first sale. Used to identify bottlenecks and enforce SLA compliance across Hunting and SOB channels.

**Sources:** Full ZOHO CRM pipeline (Accounts, Deals, Onboarding, Activations, Photography, QA Readiness, Contacts) + Hunting tables.

---

## SQL Engineering Practices

All tasks were written following the standards established by César Morales and Luis Zavala:

**Task structure:**
```sql
-- Every task follows this pattern:
-- 1. TRUNCATE target table
-- 2. INSERT INTO from source query
-- 3. Scheduled via Snowflake TASK with defined CRON
-- 4. Dependencies declared via AFTER clause in task tree

CREATE OR REPLACE TASK schema.TABLE_NAME_TASK
  WAREHOUSE = 'warehouse_name'
  SCHEDULE = 'USING CRON 30 5 * * * America/Bogota'   -- Or AFTER parent_task
AS
  INSERT INTO schema.TABLE_NAME
  WITH cte_source AS (
    SELECT ...
    FROM source_table
  )
  SELECT ...
  FROM cte_source;
```

**Key practices applied:**
- CTEs over nested subqueries for readability and maintainability
- Explicit column aliasing — no `SELECT *` in production tasks
- Timezone-aware scheduling (America/Bogota, UTC-5)
- Task tree dependencies declared explicitly via `AFTER` to guarantee execution order
- DDL documentation: `SELECT GET_DDL('task', 'schema.TASK_NAME')` used to inspect and version-control task definitions
- Error monitoring: pipeline failures tracked and escalated via Slack channel alerts

---

## Data Sources Integrated

| Source | Type | Tables |
|--------|------|--------|
| ZOHO CRM | Connector (scheduled) | 9 tables: Accounts, Deals, Onboarding, Activations, QA Readiness, Photography, Contacts, Leads, Razon Social |
| Public Microservices (CMS) | Public API tables | Stores, Brands, City Addresses, Shutdown Log, Commissions |
| Hunting | Internal | Global Activations, Signings, Closed Deals, Owners |
| Global Finances | Internal | Global Orders, Order Details, Application Users |
| Google Sheets | OAuth connector | Contactability (BPO), Hunters list, SOB Tracker, Brand Expansion |
| Microzones | Internal DS | Stores by cell, Active microzones |
| Fraud | Internal | Fraud store flags |
| Performance | Internal | Store-level full performance (TOP segmentation) |
