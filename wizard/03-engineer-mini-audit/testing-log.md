# 03 Engineer Mini-Audit — Testing Log

> **Purpose:** Step-by-step UI testing record for the Engineer Mini-Audit wizard flow
> **Live URL:** https://portal.limitlessmodus.com/engineer-mini-audit?submission_id=...
> **Code:** `limitless-portal/frontend/src/views/EngineerMiniAuditWizard.vue`
> **Instrument:** `limitless-portal/frontend/src/instruments/engineer-mini-audit.ts`

---

## Test Accounts (CREDO GROUP UK)

| # | Email | Name | Group | Assigned Audit | Section 5 Module |
|---|-------|------|-------|----------------|-----------------|
| 16 | nicholas.field@credo-group.co.uk | Nicholas Field | engineering | 03-engineer-mini-audit | TBD by Q1.2 |
| 17 | mathew.field@credo-group.co.uk | Mathew Field | engineering | 03-engineer-mini-audit | TBD by Q1.2 |

Google password for all test accounts: `CdqXDZSKDVAr2qG`

---

## Test Sessions

### Session 1 — [DATE]

**Tester:** [name]
**Account:** [email]
**Browser:** [browser + version]

*(Use the same step-by-step checklist pattern. Key additional checks:)*

#### Critical checks for Engineer Mini-Audit

- [ ] Step 1 (Q1.1): Name field is optional (can proceed without it)
- [ ] Step 1 (Q1.2): Role type selection renders 5 options including "Multi-skilled"
- [ ] Step 1 (Q1.2): Selecting "Multi-skilled" shows secondary module picker
- [ ] Step 3: TableGrid renders with Low/Med/High option (not just percentages)
- [ ] Step 3 (Q3.2): MultiSelectCapped limits to 2 selections
- [ ] Step 4 (Q4.7): FileUpload supports camera capture (mobile)
- [ ] Step 5: Correct module displays based on Q1.2 selection
- [ ] Step 5 (Multi-skilled): Both selected modules appear in sequence
- [ ] Step 6 (Q6.1): SingleSelect frequency scale renders
- [ ] Step 10: "Nothing to upload" option available
- [ ] Mobile layout: single-column, large touch targets

---

## Summary

| Step | Tested | Pass | Issues Found |
|------|--------|------|-------------|
| 0 — Consent | | | |
| 1 — About You | | | |
| 2 — What You Do | | | |
| 3 — Non-Productive Time | | | |
| 4 — Systems & Portals | | | |
| 5 — Job Type Module | | | |
| 6 — Materials & Inventory | | | |
| 7 — Quality & Safety | | | |
| 8 — Cost Leakage | | | |
| 9 — What Would You Change | | | |
| 10 — Optional Evidence | | | |
| 11 — Review & Submit | | | |
