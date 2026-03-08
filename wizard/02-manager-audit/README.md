# 02 Manager Audit — Wizard Flow

> **Instrument:** Invincibility Blueprint — Manager Audit
> **Audience:** Functional managers (all departments)
> **Steps:** 15 (Consent + Sections 1–13 + Review)
> **Time:** 30–45 minutes
> **Conditional logic:** Section 5 branches by department (selected in Section 1)

## Status

| Aspect | Status | Notes |
|--------|--------|-------|
| **Specification** | Complete | See [_shared/specification.md](../_shared/specification.md) §3 |
| **Figma layouts** | Complete | 17 frames (15 base + 2 unique Section 5 variants) |
| **Frontend instrument** | Implemented | `limitless-portal/frontend/src/instruments/manager-audit.ts` |
| **Wizard view** | Implemented | `limitless-portal/frontend/src/views/ManagerAuditWizard.vue` |
| **Conditional routing** | Implemented | Section 5 showWhen logic in wizard store |
| **Step-by-step UI testing** | Not started | See [testing-log.md](testing-log.md) |
| **Known issues** | See [issues.md](issues.md) | |

## Step Summary

| Step | Section | Questions | Key Types | Notes |
|------|---------|-----------|-----------|-------|
| 0 | Consent | 2 | ConsentCheckbox | |
| 1 | About You | 8 | FreeText, SingleSelect | **Routing trigger** (Q1.3) |
| 2 | Role Purpose | 5 | FreeTextLong, ResponseMode | |
| 3 | Regular Duties | 3 | ResponseMode, TableGrid | 3-tier capture |
| 4 | Day-in-the-Life | 17 | FreeTextLong, MultiSelect | 3 sub-groups |
| 5 | Operational Reality | Variable | Variable | **CONDITIONAL** — 9 variants |
| 6 | Non-Productive Time | 2 | TableGrid, ResponseMode | |
| 7 | Cost Leakage | 5 | FreeTextLong, ResponseMode | |
| 8 | People Management | 4 | NumberInput, FreeTextLong | Conditional on line mgr |
| 9 | Interfaces | 5 | FreeTextLong | |
| 10 | Reality Test | 2 | RatingScaleWithEvidence | 9 items, 9-point scale |
| 11 | Risks & Changes | 4 | FreeTextLong | |
| 12 | Evidence Register | 1 | ChecklistUpload (14 items) | |
| 13 | Final Catch-All | 2 | FreeTextLong | |
| 14 | Review & Submit | 0 | Summary view | |

## Section 5 Routing

| Department (Q1.3) | Variant | Questions |
|-------------------|---------|-----------|
| Field Ops – Installations | 5.1 | 7 |
| Field Ops – Service Calls/Repair | 5.2 | 6 |
| Field Ops – Pre-Enablement | 5.3 | 6 |
| Field Ops – Enablement Works | 5.4 | 6 |
| Dispatch/Scheduling | 5.5 | 8 (unique layout) |
| Stores/Materials | 5.6 + 5.6B + 5.6C | 22 (unique layout) |
| QA/Quality | 5.7 | 6 |
| HSEQ | 5.8 | 5 |
| HR | 5.9 | 5 |
| Other | — | Skip Section 5 |

## Files

| File | Purpose |
|------|---------|
| [layouts.md](layouts.md) | Per-step Figma wireframe reference with question inventories |
| [testing-log.md](testing-log.md) | Step-by-step test results and session notes |
| [issues.md](issues.md) | Open bugs and improvements |

## Related

- Specification: [_shared/specification.md](../_shared/specification.md) §3
- Figma section: "02 — Manager Audit" in Wizard — Audit Self-submission page
- Frontend code: `limitless-portal/frontend/src/instruments/manager-audit.ts`
