# 01 Company Audit — Wizard Flow

> **Instrument:** Invincibility Blueprint — Company Audit
> **Audience:** Director / MD / CEO
> **Steps:** 15 (Introduction + Consent + Company Context + Sections A–K + Evidence + Review)
> **Time:** 60–90 minutes
> **Conditional logic:** None — all sections required

## Status

| Aspect | Status | Notes |
|--------|--------|-------|
| **Specification** | Complete | See [_shared/specification.md](../_shared/specification.md) §2 |
| **Figma layouts** | Complete | 14 frames in Figma |
| **Frontend instrument** | Implemented | `limitless-portal/frontend/src/instruments/company-audit.ts` |
| **Wizard view** | Implemented | `limitless-portal/frontend/src/views/CompanyAuditWizard.vue` |
| **Step-by-step UI testing** | Not started | See [testing-log.md](testing-log.md) |
| **Known issues** | See [issues.md](issues.md) | |

## Step Summary

| Step | Section | Questions | Key Types |
|------|---------|-----------|-----------|
| 0 | Introduction | 5 | Display only |
| 1 | Consent | 4 | ConsentCheckbox |
| 2 | Company Context | 9 | FreeText, MultiSelect, TableGrid, ResponseMode |
| 3 | A — Org Foundation | 16 | FreeTextLong, ResponseMode |
| 4 | B — Org Structure | 10 | ResponseMode, ChecklistUpload, TableGrid |
| 5 | C — Workforce | 19 | FreeTextLong, ResponseMode |
| 6 | D — Strategy | 13 | FreeTextLong, SingleSelect, ResponseMode |
| 7 | E — Process & Reality | 12 | RatingScaleWithEvidence, ResponseMode |
| 8 | F — Operating Cadence | 2 | ChecklistUpload |
| 9 | G — External System | 5 | FreeTextLong + ResponseMode |
| 10 | H — Finance | 28 | FreeTextLong, ResponseMode, ChecklistUpload |
| 11 | I — Operations | 40 | FreeTextLong, TableGrid, ChecklistUpload |
| 12 | J — Leadership | 6 | FreeTextLong |
| 13 | K — Evidence Register | 3 | ResponseMode, TableGrid, FileUpload |
| 14 | Review & Submit | 0 | Summary view |

## Files

| File | Purpose |
|------|---------|
| [introduction.md](introduction.md) | Introduction step content specification |
| [layouts.md](layouts.md) | Per-step Figma wireframe reference with question inventories |
| [testing-log.md](testing-log.md) | Step-by-step test results and session notes |
| [issues.md](issues.md) | Open bugs and improvements |

## Related

- Specification: [_shared/specification.md](../_shared/specification.md) §2
- Figma section: "01 — Company Audit" in Wizard — Audit Self-submission page
- Frontend code: `limitless-portal/frontend/src/instruments/company-audit.ts`
