# Dangote Cement Plc — Digital KPI Automation & Data Quality System
### Challenge 4 | Dangote Industries Innovation Challenge 2025
**Top 20 Finalist** — National Competition

---

## Overview

This repository contains the full submission for **Challenge 4: Digital KPI Automation and Data Quality** of the Dangote Industries Innovation Challenge.

The project proposes and documents a complete **Digital KPI Reporting and Governance System** for Dangote Cement Plc, addressing systemic failures in manual daily plant performance reporting across four Nigerian plants: **Obajana**, **Ibese**, **Gboko**, and **Okpella**.

The solution is built entirely on **Microsoft Power Platform** — requiring no new vendor, no custom development stack, and no infrastructure outside of what Dangote likely already operates within Microsoft 365.

---

## The Problem

DCP's plant performance reporting depends on manual daily KPI data compiled in local Excel files and emailed to management. At 17.7Mt Nigeria volume and ₦1.6 trillion in manufacturing costs (2025), this manual process creates six compounding failures:

| Pain Point | Business Impact |
|---|---|
| Reporting delays | Management decisions made on stale data |
| Inconsistent templates across plants | Cross-plant comparison is impossible |
| Missing KPI records | Gaps in data with no audit trail |
| No validation layer | Bad numbers reach dashboards unchecked |
| Weak traceability | No record of who submitted what and when |
| No exception visibility | Issues discovered at month-end, not real-time |

---

## The Solution

A two-tier Microsoft Power Platform architecture that digitises, validates, governs, and reports KPI data across all four plants — in real time.

```
Plant Operators (all 4 plants)
         ↓
Power Apps Canvas Form
         ↓
SharePoint Lists — Tier 1 Entry Store
         ↓
Power Automate — Validate · Sign-off · Escalate
         ↓
Dataverse — Tier 2 Validated Store
         ↓
Power BI + RLS — Role-Based Dashboards
```

### Technology Stack

| Layer | Tool | Purpose |
|---|---|---|
| Data Entry | Power Apps (Canvas App) | Standardised KPI input form for plant operators |
| Workflow & Validation | Power Automate | Completeness checks, range validation, approval routing, escalation |
| Entry Storage (Tier 1) | SharePoint Lists | Low-cost raw data capture — standard M365 licence |
| Validated Storage (Tier 2) | Microsoft Dataverse | Relational store with enforced integrity and audit trail |
| Reporting & Dashboards | Power BI + RLS | Role-filtered dashboards across 5 user roles |
| Identity & Access | Microsoft 365 | Authentication via M365; RLS via email-matched user table |

---

## KPI Coverage

**16 Core KPIs tracked daily across all 4 plants:**

| Category | KPIs |
|---|---|
| Production (6) | Clinker Production Volume, Cement Production Volume, Kiln Utilization Rate, OEE, Clinker Factor, Plant Capacity Utilization Rate |
| Energy (3) | Specific Heat Consumption, Specific Power Consumption, Thermal Substitution Rate |
| Quality (3) | Free Lime Content, Blaine Fineness, Off-spec Batch Rate |
| Maintenance (3) | PM Compliance Rate, MTBF, Maintenance Cost per Tonne |
| Governance (1) | Submission Timeliness |

---

## Data Model

**Star schema — 11 tables across two storage tiers:**

```
Dimension Tables (Dataverse)          Fact Tables (SharePoint → Dataverse)
────────────────────────────          ────────────────────────────────────
DIM_Plant                             FACT_DailySubmission  ← central parent
DIM_KilnLine                          FACT_ProductionInput
DIM_User                              FACT_EnergyInput
DIM_Role                              FACT_QualityInput
                                      FACT_MaintenanceInput
                                      FACT_ValidationLog     ← system only
                                      FACT_SubmissionTimeliness ← system only
```

All relationships are Many-to-One. No Many-to-Many relationships exist in the model. Full column definitions, data types, tags (PK, FK, REQ, CALC, SYS), and relationships are documented in the Technical Appendix.

---

## Roles & Access Control

Five roles with Power BI Row-Level Security enforced via Microsoft 365 identity:

| Role | Scope | Key Access |
|---|---|---|
| Plant Operator | Own submissions only | Entry form, submission status |
| Plant Manager | Own plant only | All 15 operational KPIs, approval queue |
| Data Quality Officer | All plants — exceptions | Exception dashboard, validation flags, NCR records |
| Regional Manager | All 4 plants | Cross-plant comparison, timeliness tracking |
| Executive | All plants — strategic | Production totals, TSR, maintenance cost, EBITDA drivers |

RLS is enforced through `DIM_User[Email] = USERPRINCIPALNAME()` — no separate login required. Microsoft 365 authentication handles identity; the data model handles access control.

---

## Power Automate Workflow

The validation and governance flow runs automatically on every submission:

```
1. TRIGGER          New SharePoint item created (Power Apps submission)
2. COMPLETENESS     All required fields present? → No: notify operator, hold submission
3. RANGE VALIDATION Values within acceptable bounds? → No: log exception, flag severity
4. NCR FLAG         Quality KPI breach (Free Lime > 1.5%)? → Notify Data Quality Officer
5. DELEGATION       Plant Manager available? → No: route to designated delegate
6. APPROVAL REQUEST Send Teams + Email to Plant Manager (Approve / Reject buttons)
7. PARALLEL TIMER   T+12h: reminder → T+24h: Regional Manager escalation → T+48h: Executive escalation
8. ON APPROVAL      Promote all FACT records to Dataverse → Generate timeliness record → Notify operator
9. ON REJECTION     Notify operator with reason → Return to resubmission loop
10. SCHEDULED       At deadline: check each plant → On-time / Late / Missing → Notify if late
```

### Validation Rule Library Summary

| Category | Rules | Purpose |
|---|---|---|
| Completeness (CMP) | 18 rules | Every required field must be present |
| Range (RNG) | 17 rules | Every value must be within acceptable bounds |
| Consistency (CNS) | 6 rules | Cross-field logical checks (e.g. downtime + operating hours = 24) |
| Timeliness (TML) | 5 rules | Submission and approval SLA enforcement |

---

## Business Case

Based on Dangote Cement's 2025 full-year results:

| Metric | 2025 Value | System Connection |
|---|---|---|
| Fuel & Power Cost | ₦681.9B | Specific Heat Consumption KPI drives this directly |
| Maintenance Cost | ₦165.9B | PM Compliance + MTBF KPIs reduce unplanned downtime |
| Materials Consumed | ₦385.5B | Clinker Factor KPI controls raw material efficiency |
| Nigeria EBITDA Margin | 59.6% | All 15 operational KPIs feed into this margin |

**Conservative estimated savings:** ₦9B+ annually from energy and maintenance efficiency gains enabled by real-time KPI visibility.

**Implementation cost:** ~₦9–16M Year 1 (licensing + training + setup)

**ROI:** >500x in Year 1 on conservative assumptions.

---

## Implementation Roadmap

| Phase | Timeline | Key Actions |
|---|---|---|
| Phase 1 — Pilot | 0–30 days | Deploy at Obajana. Build Power Apps form. Set up SharePoint lists. Configure validation flow. Train 2 super-users per plant. |
| Phase 2 — Scale | 1–3 months | Roll out to all 4 plants. Deploy Power BI dashboards with RLS. Connect SharePoint → Dataverse promotion workflow. |
| Phase 3 — Optimise | 3–6 months | ERP/SAP integration. Predictive exception rules. SCADA auto-population. Mobile Power BI app for executives. |

---

## Repository Contents

```
Dangote-Cement-Digital-KPI-Automation/
│
└── Dangote Submission/
    ├── DCP_Challenge4_ExecutiveDeck.pptx        ← 10-slide executive presentation
    ├── Dangote_Challenge4_Technical_Appendix.pdf ← Full technical appendix (PDF)
    ├── Dangote_Challenge4_Technical_Appendix.docx← Full technical appendix (Word)
    └── Business_Case_Supporting.pdf              ← One-page business case
```

### Document Descriptions

**Executive Deck (PPTX)**
10-slide presentation following the prescribed challenge structure. Covers problem diagnosis, root cause hypothesis, proposed solution architecture, technical feasibility, business case with ROI, risk register, implementation roadmap, and final recommendation with pilot proposal.

**Technical Appendix (PDF/DOCX)**
Full technical documentation including: system architecture overview, complete KPI taxonomy, full data model with all 11 table schemas, ERD diagram, SharePoint list structure, Power Automate flow diagram, complete validation rule library (46 rules), Power BI RLS design, implementation prerequisites, and statement of data limitations.

**Business Case (PDF)**
One-page supporting document covering solution value, cost breakdown, key assumptions, and risk summary.

---

## Current Implementation Status

| Deliverable | Status |
|---|---|
| System architecture design | ✅ Complete |
| KPI taxonomy — 16 core KPIs | ✅ Complete |
| Data model — 11 table star schema | ✅ Complete |
| ERD diagram | ✅ Complete |
| SharePoint lists — 5 lists built | ✅ Built |
| Power Automate flow diagram | ✅ Complete |
| Validation rule library — 46 rules | ✅ Complete |
| Power Apps UI design — 4 screens | ✅ Designed |
| Executive presentation deck | ✅ Complete |
| Technical appendix | ✅ Complete |
| Business case | ✅ Complete |
| Power Apps functional build | 🔄 In progress |
| Power Automate live validation | 🔄 In progress |
| Power BI dashboard | 🔄 In progress |

> **Note on implementation status:** Full functional deployment is currently constrained by Microsoft Power Platform licensing limitations (Dataverse premium access) outside of an enterprise environment. A functional prototype covering Power Apps data entry, Power Automate validation, and Power BI reporting is being built using available Microsoft 365 access. All architecture, data model, and governance design is complete and implementation-ready.

---

## About This Project

**Competition:** Dangote Industries Innovation Challenge 2025

**Track:** Challenge 4 — Digital KPI Automation and Data Quality

**Result:** Top 20 Finalist (National)

**Plants in Scope:** Obajana (16.25 Mta) · Ibese (12 Mta) · Gboko (4 Mta) · Okpella (3 Mta)

**Submitted by:** Erioluwa Akinfisoye — 400-level Mechanical Engineering, Federal University of Technology Akure (FUTA)

This project sits at the intersection of data engineering, business intelligence, and operational governance — demonstrating that strong data analysis begins with a clean, governed, and automated data pipeline. The solution was designed to be deployable within DCP's existing Microsoft 365 infrastructure with no new vendor or custom development required.

---

## Connect

**LinkedIn:** [Erioluwa Akinfisoye](https://www.linkedin.com/in/erioluwa-akinfisoye-30533a247/)
**Email:** eriakinfisoye@gmail.com

---

*Dangote Cement Plc · Challenge 4 · Dangote Industries Innovation Challenge 2025*
