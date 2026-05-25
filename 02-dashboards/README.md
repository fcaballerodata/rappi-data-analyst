# 02 — Dashboards

## Overview

5+ Power BI Control Tower dashboards built and maintained for the Restaurant Onboarding vertical — consumed daily by BPO teams, onboarding coordinators, operations leads, and regional directors across 9 countries. Each dashboard had a defined owner, a structured tab inventory, and a staggered refresh schedule tied to upstream Snowflake task completion.

---

## Dashboard Inventory

| Dashboard | Primary Audience | Tabs | Refresh Schedule (CO time) |
|-----------|-----------------|------|---------------------------|
| Ready for Success | Onboarding leads, Regional directors | 5 | 01:00 · 07:00 · 09:00 · 14:00 · 17:00 |
| Early Success — Invisible | BPO Invisible, SOB & Hunting leads | 11 | 05:30 · 09:30 · 12:30 · 15:30 |
| QA Readiness | Onboarding coordinators, Hunters, SOB | 5 | 05:00 · 10:00 · 13:00 · 15:00 · 23:00 |
| SLAs R2S Control Tower | Operations leads, SLA monitors | 2 | 05:30 · 11:30 · 16:30 |
| SOB Control Tower | SOB team, Pre-activation BPO | 10+ | 06:30 · 10:00 · 11:30 · 14:00 · 19:00 |
| SOB Pre-activación | BPO Delta, SOB operations | 2 | 09:30 · 11:30 · 23:30 |

---

## Dashboard Detail

### 1 — Ready for Success (R2S)
**The primary pre-activation dashboard for the Onboarding vertical.**

Tracks store readiness metrics across all channels (Hunting, Brand Expansion, SOB) before a store's first sale. Also shows table update status — a critical operational indicator for the data team.

**Source tables:**
- `RFS_AVAILABILITY_30D` — main analytical table
- `RFS_TABLES_UPDATE_VW` — pipeline health monitor

**Tabs:**

| Tab | Content |
|-----|---------|
| Actualización Tablas | Last refresh timestamp per Snowflake table — pipeline health view |
| R2S Metrics Pt.1 | Pre-activation KPIs: readiness rate, credential delivery, schedule compliance |
| R2S Metrics Pt.2 | Product health: menu completeness, photo coverage, product categories |
| R2S Metrics Pt.3 | Store availability: hours configured, availability % |
| R2S Metrics Pt.4 | Conversion metrics: store-level conversion rates vs. benchmarks |

**Key KPIs:** R2S Rate · Credential Delivery % · Schedule Compliance % · Product Completeness % · Photo Coverage % · Availability % · Conversion Rate

---

### 2 — Early Success — Invisible
**BPO Invisible contactability and activation tracking dashboard.**

Used by BPO Invisible agents and their managers to prioritize which stores to contact, track contactability outcomes, and monitor agent performance. One of the most operationally intensive dashboards — refreshed 4x daily.

**Source tables:**
- `RFS_AVAILABILITY_30D`
- `RAPPI_AS_CONTACTABILITY`
- `RFS_PRODUCT_STOCKOUTS`
- `CHROME_EXTENSION_F12W`

**Tabs:**

| Tab | Content |
|-----|---------|
| Overview | Global contactability KPIs across all channels |
| Hunting — Priority Invisible | Priority store list for BPO Invisible (Hunting channel) |
| Hunting — Priority Invisible EXP | Expanded view with additional store attributes |
| SOB — Priority Invisible | Priority store list for SOB channel |
| SOB — Priority Invisible EXP | Expanded SOB view |
| Farmers EXP | Farmer channel store tracking |
| Overview Contactability | Contact attempt rates, answer rates, outcomes |
| Contactability — Per Country | Breakdown by country (9 countries) |
| Contactability — Facturación | Billing/invoicing contact tracking |
| Agents Performance | Individual agent KPIs: contacts made, success rate, response rate |
| Glosario | Term definitions for BPO agents |

**Key KPIs:** Contact Attempt Rate · Answer Rate · Activation Rate · Agent Productivity · Stockout Rate per Store

---

### 3 — QA Readiness
**Quality assurance tracking for store credential delivery across all channels.**

Tracks Initial Readiness (IR), QA Readiness for Hunting (HT) and SOB, and serves as the data source for credential delivery communications. Used by three teams simultaneously with different tab focus.

**Source tables:**
- `QAR_HT` · `QAR_SOB` · `QAR_IR`
- `RFS_CREDENTIALS_HUNTING`
- Google Sheets: Hunters list · Brand Expansion list

**Tabs:**

| Tab | Content |
|-----|---------|
| QA Readiness HT | Hunting channel QA status per store — credential eligibility |
| QA Readiness SOB | Self-Onboarding QA status per store |
| Initial Readiness | IR status tracker — pre-QA compliance check |
| IR — Productividad | IR team productivity metrics |
| Base Credentials HUNT | Store list used for Hunting credential delivery communications |

**Key KPIs:** R2S_V3 Rate · Credential Delivery % · IR Compliance % · Pending Credentials count · QA Pass Rate by channel

---

### 4 — SLAs R2S Control Tower
**Onboarding stage timing — where time is being lost.**

Tracks the time spent at each stage of the onboarding funnel from signing to first sale. Used to identify bottlenecks, enforce SLA commitments, and report to leadership on process efficiency.

**Source tables:**
- `RFS_ONB_SLA`

**Tabs:**

| Tab | Content |
|-----|---------|
| SLAs Generales | Global SLA compliance — average days per stage across all stores |
| SLAs Particulares | Store-level SLA breakdown — individual timing per store |

**Key KPIs:** Days Signing → Onboarding Start · Days Onboarding → QA · Days QA → Credentials · Days Credentials → First Sale · SLA Breach Rate

---

### 5 — SOB Control Tower
**The most comprehensive Self-Onboarding tracking dashboard.**

The operational nerve center for the SOB (Self-Onboarding) channel. Tracked the full funnel from lead to first sale across daily, weekly, and monthly granularities. Used by the SOB team, pre-activation BPO, and regional operations leadership.

**Source tables:**
- `RFS_SOB_COHORTS_ACTIVATION` · `RFS_AVAILABILITY_30D`
- `RFS_SOB_FUNNEL_DAILY/WEEKLY/MONTHLY`
- `RFS_SOB_BASE_LEADS` · `RFS_CREDENTIALS_SOB`
- `RFS_SOB_ZOHO_LEADS_PER_HOUR`
- Google Sheets: SOB Official Tracker

**Tabs:**

| Tab | Content |
|-----|---------|
| Overview | Global SOB KPIs — funnel summary |
| Daily / Weekly / Monthly | Multi-granularity funnel views |
| WTD / MTD | Week-to-date and month-to-date tracking |
| Base Activaciones 1, 2, 3 | Store activation lists by cohort |
| Base MX | Mexico-specific store base |
| Base Leads | Full lead base with status |
| Cohorts Activation | W1/W2/W3/W4 activation cohort tracking |
| Metrics Before Sale | Pre-first-sale store health indicators |
| Credentials SOB | SOB credential delivery tracker |
| VL per Hour | Lead velocity — leads entering per hour (real-time operational view) |

**Key KPIs:** Lead → Activation Rate · First Login % · Credential Delivery % · W1/W2/W3/W4 Activation Rates · Lead Velocity per Hour · MTD/WTD vs. Target

---

### 6 — SOB Pre-activación
**BPO Delta contact list — pre-activation outreach dashboard.**

Provides the pre-activation contact base for BPO Delta: stores that are self-onboarded and ready-to-sell but haven't made their first sale yet. Refreshed 3x daily to keep the BPO's outreach list current.

**Source tables:**
- `S4S_PREACTIVACION`
- `RFS_AVAILABILITY_30D`

**Tabs:**

| Tab | Content |
|-----|---------|
| Preactivación | Store list with pre-activation status for BPO Delta |
| Métricas | Pre-activation KPIs: stores contacted, conversion rate, pending outreach |

---

## Operational Standards Applied

| Standard | Implementation |
|----------|---------------|
| Refresh scheduling | All dashboards staggered to start after upstream Snowflake tasks complete |
| Data freshness | Last refresh timestamp visible on primary tabs |
| Pipeline monitoring | "Actualización Tablas" tab in R2S shows last update time per source table |
| Glosario | Term definitions included in operationally complex dashboards (e.g., Invisible) |
| Audience segmentation | Each dashboard documented with primary user and their specific tab focus |
