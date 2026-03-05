# Self-submission Audit Wizard — Specification

> **Version:** 1.0 (2 Mar 2026)
> **Status:** Draft — for review before implementation
> **Target repository:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal)
> **First engagement:** Nano Fibre UK (27 participants, audit go-live 10 Mar 2026)

---

## Contents

1. [Overview and Design Principles](#1-overview-and-design-principles)
2. [Wizard Step Flow — 01 Company Audit](#2-wizard-step-flow--01-company-audit)
3. [Wizard Step Flow — 02 Manager Audit](#3-wizard-step-flow--02-manager-audit)
4. [Wizard Step Flow — 03 Engineer Mini-Audit](#4-wizard-step-flow--03-engineer-mini-audit)
5. [Conditional Routing Rules](#5-conditional-routing-rules)
6. [Data Model](#6-data-model)
7. [Authentication and Authorisation](#7-authentication-and-authorisation)
8. [Participant Journey](#8-participant-journey)
9. [Admin Dashboard Requirements](#9-admin-dashboard-requirements)
10. [File Upload and Evidence Management](#10-file-upload-and-evidence-management)
11. [Auto-save, Progress Tracking, and Submission Lifecycle](#11-auto-save-progress-tracking-and-submission-lifecycle)
12. [API Contract Outline](#12-api-contract-outline)
13. [Appendix A — Nano Fibre UK Engagement Configuration](#appendix-a--nano-fibre-uk-engagement-configuration)

---

## 1. Overview and Design Principles

### 1.1 What This Specifies

This document defines the interactive web-based wizard that delivers the three Invincibility Blueprint self-submission audit instruments:

| # | Instrument | Audience | Wizard Steps | Time | Conditional Logic |
|---|-----------|----------|-------------|------|-------------------|
| 01 | Company Audit | Director / MD | 13 (Consent + 1 + A–K) | 60–90 min | None — all sections required |
| 02 | Manager Audit | Functional managers | 15 (Consent + 1–13) | 30–45 min | Section 5 routes by department |
| 03 | Engineer Mini-Audit | Field engineers | 12 (Consent + 1–10) | 10–12 min | Section 5 routes by job type |

The wizard replaces Greg's current Word docs + Microsoft Forms delivery with an interactive, auto-saving, section-by-section web experience.

### 1.2 Design Principles

1. **Speed and truth, not presentation** — the UI must make it faster to submit than to format a Word document. Pre-populated fields, clear skip paths, and auto-save reduce friction.
2. **Section-at-a-time** — each wizard step maps to one section of the instrument. Participants see progress and can return to any completed section.
3. **ResponseMode pattern** — for every evidence/upload request, offer the four response modes: Upload / Provide link / Not available (covered in interviews) / Will send later (ETA).
4. **Mobile-aware** — field engineers will likely complete the Mini-Audit on phones or tablets. The wizard must be responsive.
5. **No blame, evidence over polish** — the UI reinforces this message at every step. No red validation errors for incomplete sections — just a "complete" vs "in progress" indicator.
6. **Draft persistence** — every field auto-saves. Participants can close and resume at any point.
7. **Confidentiality first** — consent gate before any data entry. Upload guidance shown before every file input.

### 1.3 Question Type Taxonomy

Every question across all three instruments maps to one of these input types:

| Type | Widget | Validation | Example |
|------|--------|-----------|---------|
| `FreeText` | Single-line `<input>` or multi-line `<textarea>` | Optional min/max length | "Legal entity name + trading name" |
| `FreeTextLong` | Multi-line `<textarea>` with rich-text option | Optional min length | "Describe your operational system" |
| `SingleSelect` | Radio buttons or dropdown | Must select one | OD Stage self-assessment (1–4) |
| `MultiSelect` | Checkbox group | Optional min/max selections | "Primary services delivered (tick all)" |
| `MultiSelectCapped` | Checkbox group with max constraint | Max N selections | "Pick top 2 categories" |
| `FileUpload` | Drag-and-drop file zone + metadata | Max file size, allowed types | "Upload org chart" |
| `ResponseMode` | 4-option radio group + conditional sub-fields | Must select one mode | "Upload what we have / Provide link / Not available / Will send later" |
| `TableGrid` | Structured table with editable cells | At least 1 row | Non-productive time estimates, Regular Duties cadence |
| `RatingScale` | Dropdown or radio per item (8-point scale) | Must rate each standard item | Reality Test ratings |
| `RatingScaleWithEvidence` | Rating dropdown + FreeText + optional FileUpload per item | Rating required, evidence optional | Reality Test with evidence description |
| `ConsentCheckbox` | Single checkbox | Must be checked to proceed | GDPR consent |
| `NumberInput` | Numeric input with optional unit | Optional range | "How many disciplinaries are open?" |
| `ChecklistUpload` | Checkbox list where each item has a ResponseMode sub-field | At least awareness (check or skip) | B2.2 department-by-department uploads |

---

## 2. Wizard Step Flow — 01 Company Audit

**Audience:** Director / MD / CEO
**Total steps:** 13 (Consent + Company Context + Sections A–K)
**Estimated time:** 60–90 minutes
**Conditional logic:** None — all sections required

### Step 0: Consent

| # | Question | Type | Required |
|---|----------|------|----------|
| 0.1 | Confidentiality statement (read-only display) | — | Display only |
| 0.2 | Upload guidance (read-only display) | — | Display only |
| 0.3 | "Before You Start" guidance (read-only display) | — | Display only |
| 0.4 | Consent checkbox: "I confirm I am authorised..." | `ConsentCheckbox` | Yes — gate |

**Gate:** Cannot proceed to Step 1 until consent is checked.

### Step 1: Company Context

| # | Question | Type | Required |
|---|----------|------|----------|
| 1.1 | Legal entity name + trading name | `FreeText` | Yes |
| 1.2 | Primary services delivered | `MultiSelect` | Yes |
|  | Options: Broadband installations / Service calls / Pre-Enablement / Enablement works | | |
| 1.3 | Clients: network operators/ISPs you deliver for | `FreeTextLong` | Yes |
| 1.4 | Geographic footprint: regions, depots/stores coverage | `FreeTextLong` | Yes |
| 1.5 | Headcount & workforce mix (FTE vs contractors, by function) | `FreeTextLong` | Yes |
| 1.6 | Volume profile (last 12 weeks) | `TableGrid` | Yes |
|  | Columns: Workstream / Volume per week | | |
|  | Rows: Installs / Service calls / Pre-enablement / Enablement | | |
| 1.7 | Top 5 priorities (next 12 months) | `FreeTextLong` | Yes |
| 1.8 | Company overview pack | `ResponseMode` | No |
| 1.9 | Catch-all: what else exists (Context) | `FreeTextLong` + `FileUpload` | No |

### Step 2: Section A — Organisational Foundation

| # | Question | Type | Required |
|---|----------|------|----------|
| A1.1 | Organisation purpose statement (1 paragraph) | `FreeTextLong` | Yes |
| A1.2 | Core values + delivery behaviours | `FreeTextLong` | Yes |
| A1.3 | Non-negotiables (behaviours not tolerated) | `FreeTextLong` | Yes |
| A1.4 | Purpose/values documents | `ResponseMode` | No |
| A2.1 | Operational system description | `FreeTextLong` | Yes |
| A2.2 | 3–5 stability rules | `FreeTextLong` | Yes |
| A2.3 | Operating model/playbook | `ResponseMode` | No |
| A3.1 | Org chart / organogram | `ResponseMode` | Yes |
| A3.2 | Top-level functions list | `FreeTextLong` | Yes |
| A4.1 | Financial approvals thresholds | `FreeTextLong` | Yes |
| A4.2 | Resourcing approvals | `FreeTextLong` | Yes |
| A4.3 | Hiring/firing authority | `FreeTextLong` | Yes |
| A4.4 | Contract/commercial approvals | `FreeTextLong` | Yes |
| A4.5 | HSEQ stop-work authority | `FreeTextLong` | Yes |
| A4.6 | Escalation thresholds | `FreeTextLong` | Yes |
| A4.7 | DoA matrix / decision schedule | `ResponseMode` | Yes |
| A.catch | Catch-all (Foundation) | `FreeTextLong` + `FileUpload` | No |

### Step 3: Section B — Organisation Structure, Departments & Roles

| # | Question | Type | Required |
|---|----------|------|----------|
| B1.1 | Latest organisation chart | `ResponseMode` | Yes |
| B1.2 | Departments/teams list (purpose, key outputs, interfaces) | `FreeTextLong` | Yes |
| B1.3 | Team charters / department summaries | `ResponseMode` | No |
| B2.1 | Regular Duties guidance (read-only display) | — | Display only |
| B2.2 | Department-by-department uploads | `ChecklistUpload` | Yes |
|  | Items: Installs / Service / Pre-Enablement / Enablement / Dispatch / Stores / QA / HSEQ / Client Performance / Support functions / Custom | | |
|  | Each item: `ResponseMode` (Upload / Link / Not available / Will send later) | | |
| B3.1 | Role descriptions / job descriptions | `ResponseMode` | Yes |
| B3.2 | Per management layer: role purpose, top 5 outcomes, key decisions | `FreeTextLong` | Yes |
| B4.1 | Critical Outcomes Ownership (10 items) | `TableGrid` | Preferred |
|  | Columns: Outcome / Owner / How measured / Current performance | | |
|  | Rows: 10 pre-populated outcomes | | |
| B.catch | Catch-all (Roles) | `FreeTextLong` + `FileUpload` | No |

### Step 4: Section C — Workforce, Capability, Performance & Discipline

| # | Question | Type | Required |
|---|----------|------|----------|
| C1.1 | Headcount by department/function | `FreeTextLong` | Yes |
| C1.2 | Attrition last 12 months + top 3 reasons | `FreeTextLong` | Yes |
| C1.3 | Contractor reliance % | `FreeTextLong` | Yes |
| C1.4 | Anonymised headcount report | `ResponseMode` | Preferred |
| C2.1 | Mandatory competencies/accreditations | `FreeTextLong` | Yes |
| C2.2 | How competence is tracked/refreshed/audited | `FreeTextLong` | Yes |
| C2.3 | Competence matrix / training records | `ResponseMode` | Preferred |
| C3.1 | Performance measurement approach | `FreeTextLong` | Yes |
| C3.2 | Last 3 months performance reporting | `ResponseMode` | Yes |
| C4.1 | Open disciplinaries breakdown | `FreeTextLong` | Yes |
| C4.2 | Top 3 disciplinary themes | `FreeTextLong` | Yes |
| C4.3 | Time-to-close and where disciplinaries stall | `FreeTextLong` | Yes |
| C4.4 | Anonymised disciplinary summary | `ResponseMode` | Preferred |
| C5.1 | Succession plan for critical roles | `FreeTextLong` | No |
| C5.2 | Where capability breaks if you lose 2 key people | `FreeTextLong` | No |
| C5.3 | Succession plan upload | `ResponseMode` | No |
| C6.1 | Bonus/reward scheme description | `FreeTextLong` | No |
| C6.2 | Where incentives drive wrong behaviours | `FreeTextLong` | No |
| C.catch | Catch-all (People) | `FreeTextLong` + `FileUpload` | No |

### Step 5: Section D — Strategy, Planning & Execution Discipline

| # | Question | Type | Required |
|---|----------|------|----------|
| D1.1 | Current strategy description | `FreeTextLong` | Yes |
| D1.2 | Top 3 constraints blocking strategy | `FreeTextLong` | Yes |
| D1.3 | Strategy deck / business plan | `ResponseMode` | No |
| D2.1 | Top company goals (3–10) with success criteria, owner, status, dates | `FreeTextLong` | Yes |
| D2.2 | Key objectives/projects per goal | `FreeTextLong` | Yes |
| D2.3 | Top 3 active objectives task plan | `FreeTextLong` | Yes |
| D2.4 | Top 10 overdue tasks/actions and why | `FreeTextLong` | Yes |
| D2.5 | Where plans typically fail | `FreeTextLong` | Yes |
| D2.6 | Goals/project/task trackers | `ResponseMode` | Preferred |
| D3.1 | OD Stage self-assessment | `SingleSelect` | Yes |
|  | Options: Stage 1 (Start-up) / Stage 2 (Regular Management) / Stage 3 (Viral Expansion) / Stage 4 (Synergistic Brotherhood) | | |
| D3.2 | Evidence for selected stage (3–5 examples) | `FreeTextLong` | Yes |
| D3.3 | Target stage in 12–24 months and why | `FreeTextLong` | Yes |
| D.catch | Catch-all (Strategy/Plans) | `FreeTextLong` + `FileUpload` | No |

### Step 6: Section E — Process, Compliance & Reality Tests

| # | Question | Type | Required |
|---|----------|------|----------|
| E1.1 | Process map: Installs | `FreeTextLong` + `ResponseMode` | Yes |
| E1.2 | Process map: Service | `FreeTextLong` + `ResponseMode` | Yes |
| E1.3 | Process map: Pre-Enablement | `FreeTextLong` + `ResponseMode` | Yes |
| E1.4 | Process map: Enablement | `FreeTextLong` + `ResponseMode` | Yes |
| E1.5 | Top 10 break points | `FreeTextLong` | Yes |
| E1.6 | Process maps / SOP index / work instructions | `ResponseMode` | Yes |
| E2.1 | Reality Test: 10 standard components | `RatingScaleWithEvidence` | Yes |
|  | Rating options (8-point): Exists and is used / Exists but not used consistently / Claimed to exist but cannot be found / Exists but under-resourced / Implemented but no effect / Tool used for wrong purpose / Multiple approaches conflict / Works only because specific people carry it | | |
|  | Items: Planning & scheduling / Dispatch & job allocation / QA gates / Repeat faults loop / Subcontractor governance / Stores control / HSEQ controls / KPI reporting / FSM-portal integration / Portal closure rules | | |
| E2.2 | Additional components (up to 8) | `RatingScaleWithEvidence` | Yes |
| E3.1 | Contract register summary | `ResponseMode` | Yes |
| E3.2 | KPI/SLA schedules and scorecards | `ResponseMode` | Yes |
| E3.3 | Commercial governance description | `FreeTextLong` | Yes |
| E3.4 | Contract extracts (redacted) | `ResponseMode` | No |
| E.catch | Catch-all (Process/Compliance) | `FreeTextLong` + `FileUpload` | No |

### Step 7: Section F — Operating Cadence & Control Schedules

| # | Question | Type | Required |
|---|----------|------|----------|
| F.grid | 11 schedules (F1–F11) | `ChecklistUpload` | Yes |
|  | Each item: Schedule name + `ResponseMode` (Upload / None + explain) | | |
|  | F1: Reporting & communications / F2: Decision-making / F3: KPI / F4: SLA / F5: Absence cover / F6: Escalation / F7: Out-of-hours / F8: Systems / F9: Reports / F10: Documentation / F11: Customer feedback | | |
| F.catch | Catch-all (Cadence/Schedules) | `FreeTextLong` + `FileUpload` | No |

### Step 8: Section G — External System

| # | Question | Type | Required |
|---|----------|------|----------|
| G.1 | Supplier/subcontractor register | `FreeTextLong` + `ResponseMode` | Yes |
| G.2 | Client register | `FreeTextLong` + `ResponseMode` | Yes |
| G.3 | Service catalogue | `FreeTextLong` + `ResponseMode` | Yes |
| G.4 | Top 5 market/industry forces | `FreeTextLong` | Yes |
| G.catch | Catch-all (External) | `FreeTextLong` + `FileUpload` | No |

### Step 9: Section H — Finance & Full Cost Audit

| # | Question | Type | Required |
|---|----------|------|----------|
| H1.1 | Last 12 months monthly actuals P&L | `ResponseMode` | Yes |
| H1.2 | Last FY P&L + current forecast | `ResponseMode` | Yes |
| H1.3 | Chart of accounts | `ResponseMode` | No |
| H2.1 | Fleet costs | `FreeTextLong` | Yes |
| H2.2 | Tools/test equipment | `FreeTextLong` | Yes |
| H2.3 | Subcontractor costs | `FreeTextLong` | Yes |
| H2.4 | Overtime/standby/on-call | `FreeTextLong` | Yes |
| H2.5 | Rework cost indicators | `FreeTextLong` | Yes |
| H2.6 | Cost driver uploads | `ResponseMode` | No |
| H3.1 | Estates list (purpose, utilisation, cost, lease terms) | `FreeTextLong` | Yes |
| H3.2 | Leases / facilities cost schedule | `ResponseMode` | No |
| H4.1 | Where costs are leaking | `FreeTextLong` | Yes |
| H4.2 | What costs feel normal but are waste | `FreeTextLong` | Yes |
| H4.3 | Where penalties/credits/chargebacks occur | `FreeTextLong` | Yes |
| H.catch.1 | Catch-all (Finance/Costs) | `FreeTextLong` + `FileUpload` | No |
| H5.1 | Inventory value split (warehouse/depots/van/in-transit) | `FreeTextLong` | Yes |
| H5.2 | Inventory ageing by value | `FreeTextLong` | Yes |
| H5.3 | Slow-moving/obsolete % and write-off policy | `FreeTextLong` | Yes |
| H5.4 | Stock turns / Days on Hand | `FreeTextLong` | Yes |
| H5.5 | Stock accuracy (cycle counts vs book) | `FreeTextLong` | Yes |
| H5.6 | Stuck returns value and management | `FreeTextLong` | Yes |
| H5.7 | Client-owned/consignment stock rules | `FreeTextLong` | Yes |
| H5.uploads | Inventory uploads (7 items) | `ChecklistUpload` | Preferred |
| H6.1 | Unit-rate schedules / rate cards | `ResponseMode` | Yes |
| H6.2 | "Done for payment" description per client | `FreeTextLong` | Yes |
| H6.3 | Throughput pipeline (last 3 months) | `FreeTextLong` + `ResponseMode` | Yes |
| H6.4 | WIP/unbilled backlog | `FreeTextLong` | Yes |
| H6.5 | Top 10 rejected closure reasons | `FreeTextLong` | Yes |
| H6.6 | Top 10 invoice dispute reasons | `FreeTextLong` | Yes |
| H6.7 | Credits/penalties/chargebacks summary | `FreeTextLong` | Yes |
| H6.uploads | Throughput uploads (5 items) | `ChecklistUpload` | Preferred |

### Step 10: Section I — Operations Deep Dive

| # | Question | Type | Required |
|---|----------|------|----------|
| I1.1 | Install flow description | `FreeTextLong` | Yes |
| I1.2 | Top 10 install failure reasons | `FreeTextLong` | Yes |
| I1.3 | Right First Time % and measurement | `FreeTextLong` | Yes |
| I1.4 | Early Life Failure % and measurement | `FreeTextLong` | Yes |
| I1.5 | Appointment promise control | `FreeTextLong` | Yes |
| I1.6 | Where installs get blocked | `FreeTextLong` | Yes |
| I1.7 | Install uploads | `ResponseMode` | No |
| I2.1 | Service flow description | `FreeTextLong` | Yes |
| I2.2 | SLA targets vs actual | `FreeTextLong` | Yes |
| I2.3 | Repeat fault rate and causes | `FreeTextLong` | Yes |
| I2.4 | Root cause process | `FreeTextLong` | Yes |
| I2.5 | Service uploads | `ResponseMode` | No |
| I3.1 | Pre-enablement flow description | `FreeTextLong` | Yes |
| I3.2 | Common outcomes | `FreeTextLong` | Yes |
| I3.3 | Success measurement | `FreeTextLong` | Yes |
| I3.4 | Where pre-enablement creates waste | `FreeTextLong` | Yes |
| I3.5 | Pre-enablement uploads | `ResponseMode` | No |
| I4.1 | Enablement workflow description | `FreeTextLong` | Yes |
| I4.2 | % installs/service blocked by enablement | `FreeTextLong` | Yes |
| I4.3 | Common enablement types | `FreeTextLong` | Yes |
| I4.4 | Where delays occur | `FreeTextLong` | Yes |
| I4.5 | Enablement uploads | `ResponseMode` | No |
| I5.1 | Dispatch rules | `FreeTextLong` | Yes |
| I5.2 | Non-productive time breakdown | `TableGrid` | Yes |
|  | Columns: Category / Estimate | | |
|  | Rows: Van-travel / Stores collection / Waiting access / Waiting records / Waiting permits / Rework / Admin / Portal admin / FSM mismatch / Other | | |
| I5.3 | Top 5 causes of lost time per week | `FreeTextLong` | Yes |
| I5.4 | Dispatch uploads | `ResponseMode` | No |
| I6.1 | Materials flow description | `FreeTextLong` | Yes |
| I6.2 | Where losses/waste occur | `FreeTextLong` | Yes |
| I6.3 | Stockout frequency and impact | `FreeTextLong` | Yes |
| I6.4 | Top 20 SKUs by value | `FreeTextLong` | No |
| I6.5 | "Just in case" stock | `FreeTextLong` | Yes |
| I6.6 | Unusable stock value | `FreeTextLong` | No |
| I6.7 | Stores uploads | `ResponseMode` | No |
| I6B.1 | Hold free-issue stock? | `SingleSelect` (Yes/No) | Yes |
| I6B.2 | Which clients and categories | `FreeTextLong` | Conditional (Yes) |
| I6B.3 | Where held | `FreeTextLong` | Conditional (Yes) |
| I6B.4 | How tracked separately | `FreeTextLong` | Conditional (Yes) |
| I6B.5 | Reconciliation method | `FreeTextLong` | Conditional (Yes) |
| I6B.6 | Discrepancy handling | `FreeTextLong` | Conditional (Yes) |
| I6B.7 | Current discrepancy exposure | `FreeTextLong` | Conditional (Yes) |
| I6B.uploads | Free-issue uploads (5 items) | `ChecklistUpload` | No |
| I7.1 | QA gates description | `FreeTextLong` | Yes |
| I7.2 | How findings become corrective action | `FreeTextLong` | Yes |
| I7.3 | QA uploads | `ResponseMode` | No |
| I8.1 | Safety standards: embedded, audited, enforced | `FreeTextLong` | Yes |
| I8.2 | Last 6–12 months safety summary | `FreeTextLong` | Yes |
| I8.3 | HSEQ uploads | `ResponseMode` | No |
| I.catch | Catch-all (Operations) | `FreeTextLong` + `FileUpload` | No |

### Step 11: Section J — Leadership System Signals

| # | Question | Type | Required |
|---|----------|------|----------|
| J.1 | % leadership time firefighting vs prevention | `FreeTextLong` | Yes |
| J.2 | Last 5 repeat problems that keep returning | `FreeTextLong` | Yes |
| J.3 | Where important information arrives late | `FreeTextLong` | Yes |
| J.4 | Where senior leaders bypass managers | `FreeTextLong` | Yes |
| J.5 | Where trust in management system is low | `FreeTextLong` | Yes |
| J.catch | Catch-all (Leadership) | `FreeTextLong` + `FileUpload` | No |

### Step 12: Section K — Evidence Register

| # | Question | Type | Required |
|---|----------|------|----------|
| K.1 | Document register / index | `ResponseMode` | No |
| K.2 | Document list (if no register) | `TableGrid` | No |
|  | Columns: Name / Owner / Location-link / Last updated / What it proves | | |
| K.3 | Evidence pack upload | `FileUpload` | No |

---

## 3. Wizard Step Flow — 02 Manager Audit

**Audience:** Functional managers (all departments)
**Total steps:** 15 (Consent + Sections 1–13)
**Estimated time:** 30–45 minutes
**Conditional logic:** Section 5 branches by department (selected in Section 1)

### Step 0: Consent

| # | Question | Type | Required |
|---|----------|------|----------|
| 0.1 | Confidentiality statement + upload guidance + "How to Use" (read-only) | — | Display |
| 0.2 | Consent checkbox | `ConsentCheckbox` | Yes — gate |

### Step 1: Section 1 — About You

| # | Question | Type | Required |
|---|----------|------|----------|
| 1.1 | Name | `FreeText` | Yes |
| 1.2 | Role title | `FreeText` | Yes |
| 1.3 | Department/team | `SingleSelect` | Yes |
|  | Options: Field Ops – Installations / Field Ops – Service Calls/Repair / Field Ops – Pre-Enablement / Field Ops – Enablement Works / Dispatch/Scheduling / Stores/Materials / QA/Quality / HSEQ / HR / Other | | |
| 1.4 | Other department (describe) | `FreeText` | Conditional (Other) |
| 1.5 | Location/region covered | `FreeText` | Yes |
| 1.6 | Who do you report to? | `FreeText` | Yes |
| 1.7 | Team size (direct + indirect) | `FreeText` | Yes |
| 1.8 | Workforce mix | `SingleSelect` | No |
|  | Options: Employed only / Subcontractors only / Both | | |

**Routing trigger:** The value of question 1.3 determines which Section 5 sub-section(s) the participant sees.

### Step 2: Section 2 — Role Purpose & What "Good" Looks Like

| # | Question | Type | Required |
|---|----------|------|----------|
| 2.1 | Purpose of your role (1 paragraph) | `FreeTextLong` | Yes |
| 2.2 | Top 5 outcomes you are accountable for | `FreeTextLong` | Yes |
| 2.3 | What "good" looks like (targets/standards/behaviours) | `FreeTextLong` | Yes |
| 2.4 | Decisions without escalation (examples) | `FreeTextLong` | Yes |
| 2.5 | Role profile / objectives / scorecard | `ResponseMode` | No |

### Step 3: Section 3 — Regular Duties & Cadence

| # | Question | Type | Required |
|---|----------|------|----------|
| 3.1 | Regular Duties & Cadence list upload | `ResponseMode` | Yes |
| 3.2 | Quick-create duties table | `TableGrid` | Fallback |
|  | Columns: Activity / Purpose / Method / Output / Trigger / Frequency / Duration / Target day / R-A | | |
| 3.3 | Fast fallback: top 10 recurring duties | `FreeTextLong` | Fallback |

### Step 4: Section 4 — Day-in-the-Life & Time Sinks

| # | Question | Type | Required |
|---|----------|------|----------|
| 4.1 | Typical day/week (short narrative) | `FreeTextLong` | Yes |
| 4.2 | Top 5 recurring interruptions | `FreeTextLong` | Yes |
| 4.3 | Non-value-adding time | `FreeTextLong` | Yes |
| 4B.1 | Systems used | `MultiSelect` | Yes |
|  | Options: Internal FSM / Client portals / QA tools / Stock system / Mapping-GIS / Time capture / Other | | |
| 4B.2 | Client portal names (if selected) | `FreeText` | Conditional |
| 4B.3 | Where information entered more than once | `FreeTextLong` | Yes |
| 4B.4 | Where status mismatches occur | `FreeTextLong` | Yes |
| 4B.5 | Portal rules that block closure | `FreeTextLong` | Yes |
| 4B.6 | Where you lose most time | `MultiSelect` | Yes |
|  | Options: Logging in / Slow portals / Searching job details / Re-keying / Evidence upload / Error correction / Conflicting instructions | | |
| 4B.7 | Workaround when systems fail | `FreeTextLong` | Yes |
| 4B.8 | Portal guides/screenshots | `ResponseMode` | No |
| 4C.1 | What counts as "done" vs "paid" | `FreeTextLong` | Yes |
| 4C.2 | Where work gets stuck after physically done | `FreeTextLong` | Yes |
| 4C.3 | Backlog tracked? Approximate volume and ageing | `FreeTextLong` | Yes |
| 4C.4 | Top 5 reasons jobs remain stuck beyond 48 hours | `FreeTextLong` | Yes |
| 4C.5 | Single lever to increase jobs accepted/paid per day | `FreeTextLong` | Yes |
| 4C.6 | Throughput uploads | `ResponseMode` | No |

### Step 5: Section 5 — Operational Reality by Department (CONDITIONAL)

See **Section 3 — Conditional Routing Rules** below for the full routing logic.

The participant sees only the sub-section(s) matching their department selection in Step 1.

#### 5.1 Field Ops — Installations

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.1.1 | Top 10 install failure reasons | `FreeTextLong` | Yes |
| 5.1.2 | Right First Time performance view | `FreeTextLong` | Yes |
| 5.1.3 | Early Life Failure view | `FreeTextLong` | Yes |
| 5.1.4 | Where installs get blocked | `FreeTextLong` | Yes |
| 5.1.5 | Where rework is created | `FreeTextLong` | Yes |
| 5.1.6 | Throughput check: done-but-not-paid in installs | `FreeTextLong` | Yes |
| 5.1.7 | Install uploads | `ResponseMode` | No |

#### 5.2 Field Ops — Service Calls / Repair

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.2.1 | Top 10 fault types | `FreeTextLong` | Yes |
| 5.2.2 | Top 5 repeat fault causes | `FreeTextLong` | Yes |
| 5.2.3 | Where fault-to-fix stalls | `FreeTextLong` | Yes |
| 5.2.4 | Evidence needed to prove resolution | `FreeTextLong` | Yes |
| 5.2.5 | Throughput check: done-but-not-paid in service | `FreeTextLong` | Yes |
| 5.2.6 | Service uploads | `ResponseMode` | No |

#### 5.3 Field Ops — Pre-Enablement

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.3.1 | What triggers pre-enablement, common outcomes | `FreeTextLong` | Yes |
| 5.3.2 | Biggest "not ready" cause | `FreeTextLong` | Yes |
| 5.3.3 | Where it reduces/creates waste | `FreeTextLong` | Yes |
| 5.3.4 | Evidence/info for install handover | `FreeTextLong` | Yes |
| 5.3.5 | Throughput check: done-but-not-paid | `FreeTextLong` | Yes |
| 5.3.6 | Pre-enablement uploads | `ResponseMode` | No |

#### 5.4 Field Ops — Enablement Works

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.4.1 | Top 10 enablement work types | `FreeTextLong` | Yes |
| 5.4.2 | Where delays occur most | `FreeTextLong` | Yes |
| 5.4.3 | % reactive firefighting vs planned | `FreeTextLong` | Yes |
| 5.4.4 | Biggest rework drivers | `FreeTextLong` | Yes |
| 5.4.5 | Throughput check: done-but-not-paid | `FreeTextLong` | Yes |
| 5.4.6 | Enablement uploads | `ResponseMode` | No |

#### 5.5 Dispatch / Scheduling

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.5.1 | Scheduling approach | `FreeTextLong` | Yes |
| 5.5.2 | Top 5 causes of schedule instability | `MultiSelect` + `FreeTextLong` per item | Yes |
|  | Options: Cancellations / Overruns / Priority-SLA escalations / Skills mismatch / Portal-driven changes / FSM-portal sync delays | | |
| 5.5.3 | Job allocation rules and portal overrides | `FreeTextLong` | Yes |
| 5.5.4 | Where dispatch loses most time | `MultiSelect` + `FreeTextLong` per item | Yes |
|  | Options: Missing job data / Portal admin / Re-keying / Chasing updates / Evidence requirements / Preventing disputes | | |
| 5.5.5 | Most common reason jobs fail to close | `FreeTextLong` | Yes |
| 5.5.6 | One system improvement wished for | `FreeTextLong` | Yes |
| 5.5.7 | Throughput check: done-but-not-paid backlog | `FreeTextLong` | Yes |
| 5.5.8 | Dispatch uploads | `ResponseMode` | No |

#### 5.6 Stores / Materials (+ 5.6B + 5.6C)

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.6.1 | Materials flow description | `FreeTextLong` | Yes |
| 5.6.2 | Where losses/waste occur | `FreeTextLong` | Yes |
| 5.6.3 | How often stockouts block jobs | `FreeTextLong` | Yes |
| 5.6.4 | Root cause of "right kit not on van" | `FreeTextLong` | Yes |
| 5.6.5 | Stores uploads | `ResponseMode` | No |
| 5.6B.1 | Largest bucket of cash tied up in stock | `FreeTextLong` | Yes |
| 5.6B.2 | Overstocked items and why | `FreeTextLong` | Yes |
| 5.6B.3 | Understocked items and impact | `FreeTextLong` | Yes |
| 5.6B.4 | Stock ageing/slow movers visibility | `FreeTextLong` | Yes |
| 5.6B.5 | Returns loop: where it stalls | `FreeTextLong` | Yes |
| 5.6B.6 | Biggest driver of stock inaccuracies | `FreeTextLong` | Yes |
| 5.6B.7 | What to change to release cash | `FreeTextLong` | Yes |
| 5.6B.8 | Inventory uploads | `ResponseMode` | No |
| 5.6C.1 | Which clients provide free-issue stock | `FreeTextLong` | Yes |
| 5.6C.2 | How client expectations are known | `FreeTextLong` | Yes |
| 5.6C.3 | Reconciliation frequency and ownership | `FreeTextLong` | Yes |
| 5.6C.4 | Top 5 discrepancy causes | `FreeTextLong` | Yes |
| 5.6C.5 | Current alignment state | `SingleSelect` | Yes |
|  | Options: Fully aligned / Minor discrepancies / Major discrepancies / Unknown | | |
| 5.6C.6 | Discrepancy handling in practice | `FreeTextLong` | Yes |
| 5.6C.7 | Free-issue uploads | `ResponseMode` | No |

#### 5.7 QA / Quality

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.7.1 | QA gates description | `FreeTextLong` | Yes |
| 5.7.2 | Top 10 recurring defects | `FreeTextLong` | Yes |
| 5.7.3 | How findings become corrective action | `FreeTextLong` | Yes |
| 5.7.4 | What prevents built-in quality | `FreeTextLong` | Yes |
| 5.7.5 | Throughput check: where quality creates done-but-not-paid | `FreeTextLong` | Yes |
| 5.7.6 | QA uploads | `ResponseMode` | No |

#### 5.8 HSEQ

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.8.1 | How safety standards are embedded/audited/enforced | `FreeTextLong` | Yes |
| 5.8.2 | Stop-work authority: clear and used? | `FreeTextLong` | Yes |
| 5.8.3 | Where incidents/near-misses cluster | `FreeTextLong` | Yes |
| 5.8.4 | Top 5 HSEQ risks | `FreeTextLong` | Yes |
| 5.8.5 | HSEQ uploads | `ResponseMode` | No |

#### 5.9 HR

| # | Question | Type | Required |
|---|----------|------|----------|
| 5.9.1 | Hardest roles to hire and why | `FreeTextLong` | Yes |
| 5.9.2 | Onboarding and competence verification | `FreeTextLong` | Yes |
| 5.9.3 | Where disciplinaries stall | `FreeTextLong` | Yes |
| 5.9.4 | Top 5 people risks | `FreeTextLong` | Yes |
| 5.9.5 | HR uploads | `ResponseMode` | No |

### Step 6: Section 6 — Non-Productive Time

| # | Question | Type | Required |
|---|----------|------|----------|
| 6.1 | Time loss estimates | `TableGrid` | Yes |
|  | Columns: Category / Estimate | | |
|  | Rows: Van-travel / Stores collection / Waiting access / Waiting records / Waiting permits / Rework / Admin / Portal admin / FSM mismatch / Other | | |
| 6.2 | Time capture uploads | `ResponseMode` | No |

### Step 7: Section 7 — Cost Leakage

| # | Question | Type | Required |
|---|----------|------|----------|
| 7.1 | Top 5 avoidable costs in your area | `FreeTextLong` | Yes |
| 7.2 | What spend feels normal but is waste | `FreeTextLong` | Yes |
| 7.3 | Where penalties/chargebacks/credits arise | `FreeTextLong` | Yes |
| 7.4 | Portal/commercial rules creating avoidable cost | `FreeTextLong` | Yes |
| 7.5 | Cost leakage uploads | `ResponseMode` | No |

### Step 8: Section 8 — People Management & Disciplinaries

| # | Question | Type | Required |
|---|----------|------|----------|
| 8.1 | Open disciplinaries count | `NumberInput` | Yes (if line mgr) |
| 8.2 | Where disciplinaries stall | `FreeTextLong` | Yes (if line mgr) |
| 8.3 | Top 3 conduct/performance themes | `FreeTextLong` | Yes (if line mgr) |
| 8.4 | What support/tools would help | `FreeTextLong` | No |

### Step 9: Section 9 — Interfaces, Handoffs & Conflicts

| # | Question | Type | Required |
|---|----------|------|----------|
| 9.1 | 3 teams you depend on most + what you need | `FreeTextLong` | Yes |
| 9.2 | 3 teams that depend on you + what they need | `FreeTextLong` | Yes |
| 9.3 | Where handoffs break and consequences | `FreeTextLong` | Yes |
| 9.4 | One interface/handoff to fix tomorrow | `FreeTextLong` | Yes |
| 9.5 | Where system handoffs break | `FreeTextLong` | Yes |

### Step 10: Section 10 — Reality Test

| # | Question | Type | Required |
|---|----------|------|----------|
| 10.1 | Standard items rating (9 items) | `RatingScaleWithEvidence` | Yes |
|  | Rating options (9-point): Exists and is used / Exists but not consistently / Claimed but can't be found / Exists but under-resourced / Implemented but no effect / Tool used for wrong purpose / Conflicting approaches / Works only because key people / Not applicable | | |
|  | Items: Planning & scheduling / Dispatch rules / QA gates / Repeat fault loop / Stores control / HSEQ controls / KPI reporting / FSM-portal integration / Portal closure rules | | |
| 10.2 | Additional items (up to 5) | `RatingScaleWithEvidence` | No |

### Step 11: Section 11 — Risks, Constraints & What You Would Change

| # | Question | Type | Required |
|---|----------|------|----------|
| 11.1 | Top 5 constraints blocking performance | `FreeTextLong` | Yes |
| 11.2 | Risks you worry about most | `FreeTextLong` | Yes |
| 11.3 | One change without extra headcount | `FreeTextLong` | Yes |
| 11.4 | One investment you would request | `FreeTextLong` | Yes |

### Step 12: Section 12 — Evidence Uploads

| # | Question | Type | Required |
|---|----------|------|----------|
| 12.1 | Evidence upload checklist | `ChecklistUpload` | Encouraged |
|  | Items (14): Local trackers / KPI packs / Rotas / Checklists-SOPs / Meeting notes / Audit results / Exception lists / Portal guides / Job pack requirements / Rejection reasons / Reconciliation trackers / Free-issue statements / Inventory reports / Other | | |

### Step 13: Section 13 — Final Catch-All

| # | Question | Type | Required |
|---|----------|------|----------|
| 13.1 | What else exists that would help us | `FreeTextLong` | Yes |
| 13.2 | Anything we should not misunderstand | `FreeTextLong` | No |

---

## 4. Wizard Step Flow — 03 Engineer Mini-Audit

**Audience:** Field engineers (all streams)
**Total steps:** 12 (Consent + Sections 1–10)
**Estimated time:** 10–12 minutes
**Conditional logic:** Section 5 routes by job type (selected in Section 1); multi-skilled engineers answer up to 2 modules

### Step 0: Consent

| # | Question | Type | Required |
|---|----------|------|----------|
| 0.1 | Confidentiality + purpose statement (read-only) | — | Display |
| 0.2 | Consent checkbox | `ConsentCheckbox` | Yes — gate |

### Step 1: Section 1 — About You

| # | Question | Type | Required |
|---|----------|------|----------|
| 1.1 | Name (optional) | `FreeText` | No |
| 1.2 | Role type | `SingleSelect` | Yes |
|  | Options: Install Engineer / Service (Fault) Engineer / Pre-Enablement Engineer / Enablement Works / Multi-skilled | | |
| 1.3 | Area/region | `FreeText` | Yes |
| 1.4 | Employment type | `SingleSelect` | Yes |
|  | Options: Employed / Subcontractor / Other | | |

**Routing trigger:** Question 1.2 determines which Section 5 module(s) appear.

### Step 2: Section 2 — What You Do & What "Good" Looks Like

| # | Question | Type | Required |
|---|----------|------|----------|
| 2.1 | Purpose of your role (1 sentence) | `FreeText` | Yes |
| 2.2 | What a "good day" looks like (1–2 bullets) | `FreeTextLong` | Yes |
| 2.3 | #1 thing that stops a good day | `FreeText` | Yes |

### Step 3: Section 3 — Non-Productive Time

| # | Question | Type | Required |
|---|----------|------|----------|
| 3.1 | Time loss estimates | `TableGrid` | Yes |
|  | Columns: Category / Estimate (% or Low-Med-High) | | |
|  | Rows: Van-travel / Stores collection / Waiting access / Waiting records / Waiting permits / Rework / Admin / Portal admin / FSM mismatch / Other | | |
| 3.2 | Top 2 categories that hurt productivity most | `MultiSelectCapped` (max 2) | Yes |
| 3.3 | One time sink to remove tomorrow and why | `FreeTextLong` | Yes |

### Step 4: Section 4 — Systems & Portals Friction

| # | Question | Type | Required |
|---|----------|------|----------|
| 4.1 | Tools used | `MultiSelect` | Yes |
|  | Options: Internal FSM / Client portals / QA tool / Stock system / Other | | |
| 4.2 | Client portal names | `FreeText` | Conditional |
| 4.3 | Where same info entered twice | `FreeTextLong` | Yes |
| 4.4 | Rejection/delay causes at closure | `MultiSelect` | Yes |
|  | Options: Missing photos / Wrong codes / Test results format / Portal timeout / Access-permissions / Conflicting FSM-portal / Other | | |
| 4.5 | How often system/portal issues delay closure | `SingleSelect` | Yes |
|  | Options: Daily / Weekly / Monthly / Rarely / Never | | |
| 4.6 | What happens after rejected closure | `FreeTextLong` | Yes |
| 4.7 | Screenshot of common rejection | `FileUpload` | No |

### Step 5: Section 5 — Job Type Modules (CONDITIONAL)

See **Section 3 — Conditional Routing Rules** below.

#### 5A. Installations Module

| # | Question | Type | Required |
|---|----------|------|----------|
| 5A.1 | Top 5 install failure reasons | `MultiSelectCapped` (max 5) | Yes |
|  | Options: Access issue / Wrong-missing kit / Network not ready / Pre-enablement incomplete / Enablement-civils required / Poor job data / Unrealistic time window / Quality standard unclear / Portal evidence rules / Other | | |
| 5A.2 | What creates repeat visits most on installs | `FreeTextLong` | Yes |
| 5A.3 | One change to improve Right First Time | `FreeTextLong` | Yes |
| 5A.4 | Throughput check: done-on-site but not paid frequency and cause | `FreeTextLong` | Yes |

#### 5B. Service Calls / Fault Module

| # | Question | Type | Required |
|---|----------|------|----------|
| 5B.1 | Top 5 fault types | `FreeTextLong` | Yes |
| 5B.2 | Biggest repeat fault causes (up to 3) | `MultiSelectCapped` (max 3) | Yes |
|  | Options: Wrong diagnosis / Parts unavailable / Network issue / Evidence constraints / Customer access / Time pressure / Other | | |
| 5B.3 | Where fault-to-fix stalls most | `FreeTextLong` | Yes |
| 5B.4 | What would reduce repeat faults fastest | `FreeTextLong` | Yes |
| 5B.5 | Throughput check: done-but-not-paid | `FreeTextLong` | Yes |

#### 5C. Pre-Enablement Module

| # | Question | Type | Required |
|---|----------|------|----------|
| 5C.1 | Most common outcomes (up to 2) | `MultiSelectCapped` (max 2) | Yes |
|  | Options: Ready for install / Blocked – needs enablement / Blocked – access / Reschedule / Job data wrong / Other | | |
| 5C.2 | Biggest "not ready" cause | `FreeTextLong` | Yes |
| 5C.3 | Evidence/info for install handover | `FreeTextLong` | Yes |
| 5C.4 | Where pre-enablement reduces/creates waste | `FreeTextLong` | Yes |
| 5C.5 | Throughput check: done-but-not-paid | `FreeTextLong` | Yes |

#### 5D. Enablement Works Module

| # | Question | Type | Required |
|---|----------|------|----------|
| 5D.1 | Top 5 enablement job types | `FreeTextLong` | Yes |
| 5D.2 | Biggest delay causes (up to 3) | `MultiSelectCapped` (max 3) | Yes |
|  | Options: Permits-TM / Materials-kit / Access / Scope unclear / Coordination / Portal approvals / Other | | |
| 5D.3 | Biggest rework drivers | `FreeTextLong` | Yes |
| 5D.4 | One change to reduce cycle time | `FreeTextLong` | Yes |
| 5D.5 | Throughput check: done-but-not-paid | `FreeTextLong` | Yes |

### Step 6: Section 6 — Materials, Inventory & Free-Issue

| # | Question | Type | Required |
|---|----------|------|----------|
| 6.1 | How often missing/wrong kit causes delays | `SingleSelect` | Yes |
|  | Options: Daily / Weekly / Monthly / Rarely / Never | | |
| 6.2 | Where kit problem usually happens (up to 2) | `MultiSelectCapped` (max 2) | Yes |
|  | Options: Van stock not correct / Stores availability / Wrong pick / Returns not processed / Job pack doesn't specify / Other | | |
| 6.3 | Extra kit carried "just in case" | `FreeTextLong` | Yes |
| 6.4 | What happens to unused kit after jobs | `FreeTextLong` | Yes |
| 6.5 | Free-issue usage recording and where it goes wrong | `FreeTextLong` | Yes |
| 6.6 | Most common reason kit not returned/credited | `FreeTextLong` | Yes |

### Step 7: Section 7 — Quality & Safety

| # | Question | Type | Required |
|---|----------|------|----------|
| 7.1 | Standards clear for your jobs? | `SingleSelect` | Yes |
|  | Options: Always / Mostly / Sometimes / Rarely / Never | | |
| 7.2 | Quality miss causes (up to 3) | `MultiSelectCapped` (max 3) | Yes |
|  | Options: Rushed schedule / Standards unclear / Wrong kit / Poor job data / Portal box-ticking / Lack of training / Other | | |
| 7.3 | Able to stop work if unsafe? | `SingleSelect` | Yes |
|  | Options: Yes always / Yes usually / Sometimes / No | | |
| 7.4 | Biggest safety risk seen most often | `FreeTextLong` | Yes |

### Step 8: Section 8 — Cost Leakage & Chargebacks

| # | Question | Type | Required |
|---|----------|------|----------|
| 8.1 | Where costs leak most (top 3) | `FreeTextLong` | Yes |
| 8.2 | Portal/commercial rules creating waste | `MultiSelect` | Yes |
|  | Options: Evidence requirements / SLA clocks / Closure codes / Repeat visit definitions / No fault found handling / Other | | |
| 8.3 | One change to reduce chargebacks fastest | `FreeTextLong` | Yes |

### Step 9: Section 9 — What Would You Change?

| # | Question | Type | Required |
|---|----------|------|----------|
| 9.1 | One change (no extra headcount) to remove waste | `FreeTextLong` | Yes |
| 9.2 | One investment request and why | `FreeTextLong` | Yes |
| 9.3 | Anything we should not misunderstand | `FreeTextLong` | No |

### Step 10: Section 10 — Optional Evidence

| # | Question | Type | Required |
|---|----------|------|----------|
| 10.1 | Evidence upload | `SingleSelect` + `FileUpload` | No |
|  | Options: Portal rejection screenshot / Job-pack evidence requirements / Example of missing-incorrect data / Nothing to upload | | |

---

## 5. Conditional Routing Rules

### 5.1 Manager Audit — Section 5 Routing

The department selected in Section 1 (question 1.3) determines which Section 5 sub-section the participant completes:

| Department Selection (1.3) | Section 5 Shown | Sub-sections Included |
|---------------------------|----------------|----------------------|
| Field Ops – Installations | 5.1 | 5.1 only |
| Field Ops – Service Calls/Repair | 5.2 | 5.2 only |
| Field Ops – Pre-Enablement | 5.3 | 5.3 only |
| Field Ops – Enablement Works | 5.4 | 5.4 only |
| Dispatch/Scheduling | 5.5 | 5.5 only |
| Stores/Materials | 5.6 | 5.6 + 5.6B + 5.6C |
| QA/Quality | 5.7 | 5.7 only |
| HSEQ | 5.8 | 5.8 only |
| HR | 5.9 | 5.9 only |
| Other | None | Skip Section 5 entirely; participant notes their function in 1.4 |

**Special cases:**
- **Stores/Materials** always gets the extended sub-sections (5.6B Inventory Value, 5.6C Free-Issue).
- **"Also relevant to" cross-pollination** — the instrument says "Also relevant to: Dispatch, Stores, QA" on some sections. In the wizard, we do NOT show cross-department sections. Instead, the instrument is designed so that overlapping themes (throughput, materials, quality) appear in the shared sections (4C, 6, 7, 9, 10) that every participant answers.
- **"Other" department** — the participant skips Section 5 entirely and provides their departmental context in the catch-all sections (particularly Section 13).

### 5.2 Engineer Mini-Audit — Section 5 Routing

The role type selected in Section 1 (question 1.2) determines which Section 5 module(s) appear:

| Role Selection (1.2) | Modules Shown | Note |
|----------------------|--------------|------|
| Install Engineer | 5A only | |
| Service (Fault) Engineer | 5B only | |
| Pre-Enablement Engineer | 5C only | |
| Enablement Works | 5D only | |
| Multi-skilled | 5A + one more (participant selects) | Show secondary module picker |

**Multi-skilled routing:**
1. When "Multi-skilled" is selected, show a secondary question: "Which two work types represent most of your week? (select 2)"
2. Options: Installations / Service / Pre-Enablement / Enablement
3. The two selected modules appear in sequence as Step 5a and Step 5b.
4. If the participant only selects one, show just that module.

### 5.3 Routing Implementation

```javascript
function getSection5Steps(instrument, profileAnswers) {
  // Manager Audit — route by department
  if (instrument === "02-manager-audit") {
    const department = profileAnswers["1.3"];
    if (department === "Stores/Materials") {
      return ["5.6", "5.6B", "5.6C"];
    } else if (department === "Other") {
      return [];  // skip Section 5 entirely
    } else {
      return [DEPT_TO_SECTION[department]];
    }
  }

  // Engineer Mini-Audit — route by job type
  if (instrument === "03-engineer-mini-audit") {
    const roleType = profileAnswers["1.2"];
    if (roleType === "Multi-skilled") {
      const secondarySelections = profileAnswers["1.2b"];
      return secondarySelections.map(s => "5" + s);  // e.g. ["5A", "5B"]
    } else {
      return [ROLE_TO_MODULE[roleType]];
    }
  }

  return [];  // Company Audit has no Section 5 routing
}
```

---

## 6. Data Model

### 6.1 Entity Relationship

```
Engagement 1──* Participant
Engagement 1──* InstrumentConfig
Participant 1──* Submission
Submission 1──* SectionDraft
Submission 1──* Upload
SectionDraft 1──* Upload
```

### 6.2 Entities

#### Engagement

```json
{
  "id": "uuid",
  "client_name": "Nano Fibre UK Limited",
  "client_legal_name": "NANO FIBRE UK LIMITED",
  "supplier_legal_name": "TUMAI MANAGEMENT LTD",
  "slug": "nano-fibre-uk",
  "subdomain": "nano",
  "status": "active",
  "audit_start_date": "2026-03-10",
  "interview_start_date": "2026-03-16",
  "contact_name": "Greg Kurnikov",
  "contact_email": "greg@limitlessmodus.com",
  "created_at": "2026-03-02T00:00:00Z",
  "azure_tenant_ids": ["nano-fibre-tenant-id"],
  "allowed_email_domains": ["nanois.co.uk", "nanofibre.co.uk"]
}
```

#### Participant

```json
{
  "id": "uuid",
  "engagement_id": "uuid",
  "name": "Ashley Wass",
  "email": "ashley.w@nanofibre.co.uk",
  "role_title": "Head of Operations",
  "department": "Operations",
  "participant_group": "heads-of-department",
  "instruments_assigned": ["02-manager-audit"],
  "section5_preset": "5.1",
  "is_anonymous": false,
  "joint_submission_group": null,
  "notes": "Internal coordinator",
  "created_at": "2026-03-02T00:00:00Z"
}
```

#### Submission

```json
{
  "id": "uuid",
  "participant_id": "uuid",
  "instrument_id": "01-company-audit",
  "status": "draft",
  "consent_given": true,
  "consent_timestamp": "2026-03-10T09:15:00Z",
  "profile_answers": {
    "1.3": "Field Ops – Installations"
  },
  "section5_routing": ["5.1"],
  "progress_percent": 45,
  "sections_completed": ["consent", "1", "2"],
  "sections_total": 15,
  "first_opened_at": "2026-03-10T09:00:00Z",
  "last_saved_at": "2026-03-10T14:30:00Z",
  "submitted_at": null,
  "locked": false
}
```

**Status values:** `not_started` → `draft` → `submitted` → `locked`

#### SectionDraft

```json
{
  "id": "uuid",
  "submission_id": "uuid",
  "section_key": "A",
  "answers": {
    "A1.1": "We build and maintain broadband networks...",
    "A1.2": "Safety first, quality always, customer focus...",
    "A2.1": { "mode": "upload", "file_id": "uuid" },
    "A3.1": { "mode": "not_available", "note": "Will cover in interview" }
  },
  "is_complete": true,
  "last_saved_at": "2026-03-10T11:22:00Z"
}
```

#### Upload

```json
{
  "id": "uuid",
  "submission_id": "uuid",
  "section_key": "B",
  "question_key": "B2.2",
  "original_filename": "Installs-Regular-Duties.xlsx",
  "stored_path": "engagements/nano-fibre-uk/submissions/uuid/B/B2.2/file.xlsx",
  "mime_type": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
  "size_bytes": 45200,
  "uploaded_at": "2026-03-10T12:00:00Z",
  "metadata": {
    "what_it_proves": "Regular duties cadence for Installations team"
  }
}
```

### 6.3 Submission Lifecycle State Machine

```
                   ┌─────────────┐
                   │ not_started  │  Participant has not opened the audit
                   └──────┬──────┘
                          │ Consent given
                          v
                   ┌─────────────┐
                   │    draft     │  Sections being completed, auto-saved
                   └──────┬──────┘
                          │ All required sections answered + "Submit" clicked
                          v
                   ┌─────────────┐
                   │  submitted   │  Read-only for participant, visible to admin
                   └──────┬──────┘
                          │ Admin locks (or auto-lock after review period)
                          v
                   ┌─────────────┐
                   │   locked     │  Immutable, ready for synthesis
                   └─────────────┘

  Reverse transitions:
  - submitted → draft : Admin unlocks for amendments (logged)
  - locked → submitted : Admin unlocks (exceptional, logged)
```

### 6.4 Joint Submission (Directors)

When two directors submit jointly (e.g., Wilhelm + Dirk):
- Both Participant records reference the same `joint_submission_group` ID.
- A single Submission record is created, owned by the group.
- Both participants can edit the same submission (last-write-wins per section, with section-level locking).
- Both names appear on the submission.

---

## 7. Authentication and Authorisation

### 7.1 Azure Entra ID (SSO)

| Aspect | Detail |
|--------|--------|
| **Provider** | Microsoft Entra ID (Azure AD) |
| **Flow** | Authorization Code Flow with PKCE (Nuxt frontend) |
| **App Registration** | Multi-tenant app, restricted to configured tenant IDs + email domains |
| **Redirect URI** | `https://nano.limitlessmodus.com/auth/callback` |
| **Scopes** | `openid`, `profile`, `email` |
| **Token storage** | HTTP-only secure cookie (access token) + refresh token in server session |

### 7.2 Participant Matching

After SSO login, the portal matches the authenticated email to a Participant record:

1. User signs in via Microsoft Entra ID.
2. Portal receives `id_token` with `email` claim.
3. Backend looks up `participant.email` matching the authenticated email (case-insensitive).
4. If match found: show participant dashboard with their assigned audit(s).
5. If no match: show "You are not registered for an active engagement. Contact your coordinator."

### 7.3 Roles

| Role | Access | How Assigned |
|------|--------|-------------|
| `participant` | Own submission(s) only: view, edit, submit | Auto-matched by email |
| `admin` | All engagements, all submissions, export, unlock | Manual flag on Participant record |
| `viewer` | Read-only access to submissions + exports | Manual flag (for Greg to review) |

### 7.4 Email Domain Restrictions

Each engagement defines `allowed_email_domains`. The SSO callback validates that the authenticated email domain is in the allowed list before granting access. Admin accounts (Tumai staff) bypass domain restrictions.

---

## 8. Participant Journey

### 8.1 Invitation Flow

1. **Admin creates engagement** in the portal with participant list.
2. **Portal generates personalised invitation emails** (or admin sends manually) containing:
   - Portal URL: `https://nano.limitlessmodus.com`
   - Brief explanation of the audit process
   - Time estimate for their specific instrument
   - Deadline (audit close date)
   - Support contact
3. **Participant clicks link**, is redirected to Microsoft SSO login.
4. **After login**, portal matches email → shows dashboard.

### 8.2 Participant Dashboard

The dashboard shows:

| Element | Detail |
|---------|--------|
| **Welcome banner** | "Welcome, Ashley. Your Invincibility Blueprint audit is ready." |
| **Assigned audit card** | Instrument name, estimated time, deadline, current status |
| **Progress bar** | Sections completed / total sections |
| **"Continue" button** | Resumes from last incomplete section |
| **Section navigator** | Clickable list of all sections with status (not started / in progress / complete) |
| **Support link** | Contact coordinator |

### 8.3 Wizard Experience

1. **Step 0 (Consent)** — must be completed first. Cannot proceed without consent.
2. **Steps 1..N** — one section per step. Can navigate forward/back. Auto-saves on navigate away.
3. **Progress indicator** — top of page shows: step X of Y, section name, progress bar.
4. **Section status** — each section shows: "Not started" / "In progress" / "Complete" in the sidebar navigator.
5. **ResponseMode** — wherever evidence is requested, the four-mode radio group appears. Selecting "Upload" expands a file drop zone. Selecting "Will send later" expands an ETA text field.
6. **Review step** — after all sections, a review screen shows all answers summary with "Edit" links per section.
7. **Submit** — locks the submission. Confirmation page with timestamp and "Your submission has been received."

### 8.4 Mobile Experience (Engineer Mini-Audit)

The Engineer Mini-Audit is designed for 10–12 minutes on a phone:
- Single-column layout
- Large touch targets for radio/checkbox inputs
- Minimal typing (most questions are select-based)
- File upload simplified (camera capture for screenshots)
- Progress saved automatically — can close and resume

---

## 9. Admin Dashboard Requirements

### 9.1 Engagement Overview

| Element | Detail |
|---------|--------|
| **Engagement header** | Client name, dates, status |
| **Submission tracker table** | 27 rows (one per participant) |
| **Columns** | Name / Role / Department / Instrument / Status / % Complete / Last Activity / Actions |
| **Status badges** | Not started (grey) / In progress (amber) / Submitted (green) / Locked (blue) |
| **Filters** | By instrument type, by status, by department |

### 9.2 Submission Viewer

- Read-only view of a submitted audit
- Section-by-section navigation
- Inline display of text answers
- File download links for uploads
- Export to JSON / Markdown for AI synthesis

### 9.3 Export

| Format | Purpose |
|--------|---------|
| **JSON** | Structured data for AI synthesis pipeline |
| **Markdown** | Human-readable per-submission report |
| **CSV** | Tabular summary of all submissions (one row per participant, columns per key question) |
| **ZIP** | All uploads for a submission or engagement |

### 9.4 Admin Actions

- **Send reminder** — trigger email to participants who haven't started or are in progress
- **Unlock submission** — revert submitted/locked back to draft (logged with reason)
- **Close engagement** — prevent new submissions, lock all drafts
- **Export all** — bulk download all submissions + uploads

---

## 10. File Upload and Evidence Management

### 10.1 Upload Rules

| Aspect | Rule |
|--------|------|
| **Max file size** | 50 MB per file |
| **Max files per question** | 5 |
| **Allowed types** | PDF, DOCX, XLSX, PPTX, CSV, PNG, JPG, JPEG, GIF, BMP, TXT, ZIP |
| **Storage** | Azure Blob Storage (engagement-scoped containers) |
| **Naming** | `{engagement}/{submission_id}/{section}/{question}/{original_filename}` |
| **Virus scan** | On upload (block if detected) |
| **Upload guidance** | Shown above every FileUpload widget: minimise personal identifiers, no special category data, redact if needed |

### 10.2 ResponseMode Widget

Every evidence request uses the ResponseMode pattern:

```
┌──────────────────────────────────────────────┐
│  How would you like to provide this?          │
│                                               │
│  ○ Upload what we have          [Drop zone]   │
│  ○ Provide a link/location      [URL field]   │
│  ○ Not available yet (cover in interviews)    │
│  ○ Will send later              [ETA field]   │
└──────────────────────────────────────────────┘
```

---

## 11. Auto-save, Progress Tracking, and Submission Lifecycle

### 11.1 Auto-save

- Every field change is auto-saved after a 2-second debounce.
- Section navigation triggers an immediate save of the current section.
- Browser tab close / navigation away triggers a save (via `beforeunload`).
- Save indicator in the header: "Saved" / "Saving..." / "Save failed (retry)".
- Conflict resolution: last-write-wins per field. No concurrent editing expected (one participant per submission, except joint director submissions).

### 11.2 Progress Tracking

- **Section-level:** each section marked as Not started / In progress / Complete.
- **Complete** = all required fields in the section have a value (or ResponseMode selection).
- **Overall progress** = completed sections / total sections (percentage).
- **Progress bar** shown on dashboard and in wizard header.

### 11.3 Submission Rules

- Participant can submit when: consent given AND at least 80% of required sections are complete.
- Incomplete optional sections do not block submission.
- On submit: all section drafts are frozen, status changes to `submitted`, timestamp recorded.
- Post-submit: participant sees read-only view of their submission with a "Request amendment" button (sends email to admin).

---

## 12. API Contract Outline

### 12.1 Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/auth/login` | Redirect to Azure SSO |
| GET | `/auth/callback` | Handle SSO callback, create session |
| POST | `/auth/logout` | Destroy session |
| GET | `/auth/me` | Current user info + participant match |

### 12.2 Participant API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/participant/dashboard` | Assigned instruments, submission status |
| GET | `/api/participant/submission/{id}` | Full submission with all section drafts |
| PUT | `/api/participant/submission/{id}/section/{key}` | Save section draft (auto-save) |
| POST | `/api/participant/submission/{id}/submit` | Final submission |
| POST | `/api/participant/submission/{id}/upload` | Upload file to section/question |
| DELETE | `/api/participant/submission/{id}/upload/{uploadId}` | Remove uploaded file |

### 12.3 Admin API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/engagements` | List all engagements |
| GET | `/api/admin/engagements/{id}` | Engagement detail + participant list |
| GET | `/api/admin/engagements/{id}/submissions` | All submissions with status |
| GET | `/api/admin/submissions/{id}` | Full submission read-only |
| POST | `/api/admin/submissions/{id}/unlock` | Revert to draft |
| POST | `/api/admin/submissions/{id}/lock` | Lock submission |
| GET | `/api/admin/engagements/{id}/export` | Export all submissions (JSON/CSV/ZIP) |
| POST | `/api/admin/engagements/{id}/remind` | Send reminder emails |

### 12.4 Instrument Definition API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/instruments` | List all instrument definitions |
| GET | `/api/instruments/{id}` | Full instrument definition (steps, questions, routing) |

Instrument definitions are stored as structured JSON matching the question inventory in Sections 2–4 of this spec. The frontend renders the wizard dynamically from these definitions.

---

## Appendix A — Nano Fibre UK Engagement Configuration

### A.1 Engagement Record

| Field | Value |
|-------|-------|
| Client | Nano Fibre UK Limited |
| Slug | `nano-fibre-uk` |
| Subdomain | `nano` (→ `nano.limitlessmodus.com`) |
| Audit start | 10 March 2026 |
| Interview start | 16 March 2026 |
| Coordinator | Ashley Wass (`ashley.w@nanofibre.co.uk`) |
| Consultant | Greg Kurnikov |
| Allowed email domains | `nanois.co.uk`, `nanofibre.co.uk` |

### A.2 Participant-to-Instrument Mapping (27 participants)

#### Directors — Company Audit (01) + Director Interview (04)

| # | Name | Email | Instrument | Section 5 Preset | Joint Group |
|---|------|-------|-----------|-----------------|-------------|
| 1 | Wilhelm Strumpher | wilhelm@nanois.co.uk | 01-company-audit | N/A | directors-joint |
| 2 | Dirk Mostert | dirk@nanois.co.uk | 01-company-audit | N/A | directors-joint |

#### Heads of Department — Manager Audit (02)

| # | Name | Email | Instrument | Section 5 Preset |
|---|------|-------|-----------|-----------------|
| 3 | Andre Steyn | andre@nanofibre.co.uk | 02-manager-audit | Other (CFO) |
| 4 | Anna Hanekom | anna@nanofibre.co.uk | 02-manager-audit | Other (Finance & Payroll) |
| 5 | Ashley Wass | ashley.w@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 6 | Lisa Woods | lisa.w@nanois.co.uk | 02-manager-audit | 5.9 (HR) |
| 7 | Callum Krzysik | callum.k@nanofibre.co.uk | 02-manager-audit | 5.7 (QA/Quality) |
| 8 | Andy Green | andrew.g@nanofibre.co.uk | 02-manager-audit | 5.8 (HSEQ) |
| 9 | Roland Dumas | roland.d@nanofibre.co.uk | 02-manager-audit | Other (Fleet) |
| 10 | Stephan Theron | stephan@nanois.co.uk | 02-manager-audit | Other (Development) |
| 11 | Gavin Estehuizen | gavin.e@nanofibre.co.uk | 02-manager-audit | 5.6 (Stores/Materials) |

#### Installation Managers — Manager Audit (02)

| # | Name | Email | Instrument | Section 5 Preset |
|---|------|-------|-----------|-----------------|
| 12 | Adam Winning | adam.w@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 13 | Damon Reid | damon.r@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 14 | Learnmore Gonda | learnmore.g@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 15 | Aaron Balmer | aaron.b@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |

#### Pre-enablement & Remediations Managers — Manager Audit (02)

| # | Name | Email | Instrument | Section 5 Preset |
|---|------|-------|-----------|-----------------|
| 16 | Kevin Du Plessis | kevin.dp@nanofibre.co.uk | 02-manager-audit | 5.3 (Pre-Enablement) |
| 17 | Jordan Middleton | jordan.m@nanofibre.co.uk | 02-manager-audit | 5.3 (Pre-Enablement) |
| 18 | Ruben Swart | ruben.s@nanofibre.co.uk | 02-manager-audit | 5.3 (Pre-Enablement) |
| 19 | Hanlo Naude | hanlo.n@nanofibre.co.uk | 02-manager-audit | 5.3 (Pre-Enablement) |

#### Supervisors — Manager Audit (02)

| # | Name | Email | Instrument | Section 5 Preset |
|---|------|-------|-----------|-----------------|
| 20 | Jack Lunt | jack.l@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 21 | Calvin Chimedza | calvin.c@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 22 | Arno De Greeff | arno.dg@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |
| 23 | Micheal Hibbert | hibbert.m@nanofibre.co.uk | 02-manager-audit | 5.1 (Installations) |

#### Engineers — Engineer Mini-Audit (03)

| # | Name | Email | Instrument | Anonymous | Note |
|---|------|-------|-----------|----------|------|
| 24 | JP Rowan | john-paul.r@nanofibre.co.uk | 03-engineer-mini-audit | Optional | Interview optional |
| 25 | Nicholas Mensforth | nicholas.m@nanofibre.co.uk | 03-engineer-mini-audit | Optional | Interview optional |
| 26 | Mateusz Biernacki | Mateusz.b@nanofibre.co.uk | 03-engineer-mini-audit | Optional | Email has capital M |
| 27 | Mathew Morley | matt.m@nanofibre.co.uk | 03-engineer-mini-audit | Optional | Interview optional |

### A.3 Section 5 Preset Notes

Some participants have departments that don't map directly to a Section 5 sub-section (CFO, Finance & Payroll, Fleet, Development). These participants get `section5_preset = "Other"` and skip Section 5, answering only the universal sections (1–4, 6–13). The "Other" field in Section 1 (question 1.4) captures their functional area.

Ashley Wass (Head of Operations) is mapped to 5.1 (Installations) as her primary operational domain, consistent with her role overseeing field delivery.

Supervisors are all mapped to 5.1 (Installations) as they supervise field installation teams.

### A.4 Open Questions from Onboarding

1. **Andre Steyn (CFO)** — should he also contribute to the Company Audit alongside directors? If yes, add instrument `01-company-audit` to his record.
2. **Supervisors** — currently assigned Manager Audit. If Greg prefers a lighter touch, switch to Engineer Mini-Audit.
3. **Email case sensitivity** — Mateusz.b@nanofibre.co.uk has capital M. Portal email matching must be case-insensitive.

---

*Part of the [Invincibility Blueprint Toolkit](README.md) — [Limitless Modus](../../README.md).*
