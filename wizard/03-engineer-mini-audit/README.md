# 03 Engineer Mini-Audit — Wizard Flow

> **Instrument:** Invincibility Blueprint — Engineer Mini-Audit
> **Audience:** Field engineers (all streams)
> **Steps:** 12 (Consent + Sections 1–10 + Review)
> **Time:** 10–12 minutes
> **Conditional logic:** Section 5 routes by job type (Q1.2); multi-skilled engineers answer up to 2 modules

## Status

| Aspect | Status | Notes |
|--------|--------|-------|
| **Specification** | Complete | See [_shared/specification.md](../_shared/specification.md) §4 |
| **Figma layouts** | Complete | 12 frames |
| **Frontend instrument** | Implemented | `limitless-portal/frontend/src/instruments/engineer-mini-audit.ts` |
| **Wizard view** | Implemented | `limitless-portal/frontend/src/views/EngineerMiniAuditWizard.vue` |
| **Conditional routing** | Implemented | Section 5 showWhen logic in wizard store |
| **Step-by-step UI testing** | Not started | See [testing-log.md](testing-log.md) |
| **Known issues** | See [issues.md](issues.md) | |

## Step Summary

| Step | Section | Questions | Key Types | Notes |
|------|---------|-----------|-----------|-------|
| 0 | Consent | 2 | ConsentCheckbox | |
| 1 | About You | 4 | FreeText, SingleSelect | **Routing trigger** (Q1.2) |
| 2 | What You Do | 3 | FreeText, FreeTextLong | |
| 3 | Non-Productive Time | 3 | TableGrid, MultiSelectCapped | |
| 4 | Systems & Portals | 7 | MultiSelect, SingleSelect, FileUpload | |
| 5 | Job Type Module | Variable | MultiSelectCapped, FreeTextLong | **CONDITIONAL** — 4 modules |
| 6 | Materials & Inventory | 6 | SingleSelect, MultiSelectCapped, FreeTextLong | |
| 7 | Quality & Safety | 4 | SingleSelect, MultiSelectCapped | |
| 8 | Cost Leakage | 3 | FreeTextLong, MultiSelect | |
| 9 | What Would You Change | 3 | FreeTextLong | |
| 10 | Optional Evidence | 1 | SingleSelect + FileUpload | |
| 11 | Review & Submit | 0 | Summary view | |

## Section 5 Routing

| Role Type (Q1.2) | Module(s) | Questions |
|-------------------|-----------|-----------|
| Install Engineer | 5A | 4 |
| Service (Fault) Engineer | 5B | 5 |
| Pre-Enablement Engineer | 5C | 5 |
| Enablement Works | 5D | 5 |
| Multi-skilled | Up to 2 selected | 4–10 |

## Files

| File | Purpose |
|------|---------|
| [layouts.md](layouts.md) | Per-step Figma wireframe reference with question inventories |
| [testing-log.md](testing-log.md) | Step-by-step test results and session notes |
| [issues.md](issues.md) | Open bugs and improvements |

## Related

- Specification: [_shared/specification.md](../_shared/specification.md) §4
- Figma section: "03 — Engineer Mini-Audit" in Wizard — Audit Self-submission page
- Frontend code: `limitless-portal/frontend/src/instruments/engineer-mini-audit.ts`
- Mobile design notes in [layouts.md](layouts.md) appendix
