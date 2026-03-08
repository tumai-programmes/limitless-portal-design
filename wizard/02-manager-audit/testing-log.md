# 02 Manager Audit — Testing Log

> **Purpose:** Step-by-step UI testing record for the Manager Audit wizard flow
> **Live URL:** https://portal.limitlessmodus.com/manager-audit?submission_id=...
> **Code:** `limitless-portal/frontend/src/views/ManagerAuditWizard.vue`
> **Instrument:** `limitless-portal/frontend/src/instruments/manager-audit.ts`

---

## Test Accounts (CREDO GROUP UK)

| # | Email | Name | Group | Assigned Audit | Section 5 Variant |
|---|-------|------|-------|----------------|-------------------|
| 14 | kevin.operations@credo-group.co.uk | Kevin Operations | operations | 02-manager-audit | TBD by Q1.3 |
| 15 | damon.operations@credo-group.co.uk | Damon Operations | operations | 02-manager-audit | TBD by Q1.3 |

Google password for all test accounts: `CdqXDZSKDVAr2qG`

---

## Test Sessions

### Session 1 — [DATE]

**Tester:** [name]
**Account:** [email]
**Browser:** [browser + version]

*(Use the same step-by-step checklist pattern as 01-company-audit. Key additional checks:)*

#### Critical checks for Manager Audit

- [ ] Step 1 (Q1.3): Department selection renders all 10 options
- [ ] Step 1 (Q1.3): Selecting a department triggers Section 5 routing
- [ ] Step 1 (Q1.4): "Other" text field appears when "Other" selected
- [ ] Step 5: Correct variant displays based on Q1.3 selection
- [ ] Step 5 (Dispatch): Compound MultiSelect + FreeTextLong renders
- [ ] Step 5 (Stores): All 3 sub-sections (5.6, 5.6B, 5.6C) display
- [ ] Step 8: NumberInput renders for disciplinary count
- [ ] Step 10: RatingScaleWithEvidence renders 9 items with 9-point scale
- [ ] Step 12: ChecklistUpload renders 14 evidence categories
- [ ] Review: Section 5 shows the routed variant, not all variants

---

## Summary

| Step | Tested | Pass | Issues Found |
|------|--------|------|-------------|
| 0 — Consent | | | |
| 1 — About You | | | |
| 2 — Role Purpose | | | |
| 3 — Regular Duties | | | |
| 4 — Day-in-the-Life | | | |
| 5 — Operational Reality | | | |
| 6 — Non-Productive Time | | | |
| 7 — Cost Leakage | | | |
| 8 — People Management | | | |
| 9 — Interfaces | | | |
| 10 — Reality Test | | | |
| 11 — Risks & Changes | | | |
| 12 — Evidence Register | | | |
| 13 — Final Catch-All | | | |
| 14 — Review & Submit | | | |
