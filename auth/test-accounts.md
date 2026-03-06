# Test Accounts — Complete Inventory

All accounts for the Limitless Portal, mapped to SSO providers and database tables.

**Engagement:** Nano Fibre UK (`72ed7515-f133-469e-9e4e-ab2e5d6c9171`)

## Two Tables, Two Dashboards

| Table | Who | Dashboard | Purpose |
|-------|-----|-----------|---------|
| `users` | **Limitless Modus consultants** (Fedor, Greg) | Consultant dashboard (`/dashboard`) | Manage engagements, view all audits, findings, interviews |
| `participants` | **Client company employees** (directors, managers, engineers) | Participant dashboard (`/participant-dashboard`) | Fill in audit forms and surveys |

The SSO callback checks `participants` first, then `users`. If found in neither, the user sees "not registered".

---

## Consultants (`users` table)

These are Limitless Modus internal team members who deliver the service.

### Active

| # | Email | Name | Role | SSO Provider | Status |
|--:|-------|------|------|-------------|--------|
| 1 | `fedor.vasilyev@tumai.cc` | Fedor Vasilyev | consultant | Microsoft (tumai.cc) | In DB, tested |
| 2 | `fedor.vasilyev@tumai.com` | Fedor Vasilyev | consultant | Google (tumai.com) | In DB, tested |
| 3 | `greg.kurnikov@odexpert.co.uk` | Greg Kurnikov | consultant | Microsoft (odexpert.co.uk) | **Not in DB yet** |

### Planned (future)

| # | Email | Name | Role | SSO Provider | Status |
|--:|-------|------|------|-------------|--------|
| 4 | `fedor@limitlessmodus.com` | Fedor Vasilyev | consultant | Google (limitlessmodus.com) | Not created yet |
| 5 | `greg@limitlessmodus.com` | Greg Kurnikov | consultant | Google (limitlessmodus.com) | Not created yet |

---

## Client Test Accounts — Microsoft SSO (tumai.cc)

These simulate Nano Fibre UK client employees. All should be in the `participants` table.

| # | Email | Name | Role Title | Password | Department | Group | Status |
|--:|-------|------|-----------|----------|------------|-------|--------|
| 6 | `mark.director@tumai.cc` | Mark Director | Director | Kaxa172305 | Leadership | leadership | OK |
| 7 | `adam.finance@tumai.cc` | Adam Finance | Finance Manager | Joto995535 | Finance | finance | OK |
| 8 | `kevin.operations@tumai.cc` | Kevin Operations | Operations Manager | Bado798597 | Operations | operations | OK |
| 9 | `damon.operations@tumai.cc` | Damon Operations | Operations Supervisor | Xaqo596036 | Operations | operations | OK |
| 10 | `nicholas.field@tumai.cc` | Nicholas Field | Field Engineer | Xobo878227 | Field Engineering | engineering | OK |
| 11 | `mathew.field@tumai.cc` | Mathew Field | Field Engineer | Mucu340428 | Field Engineering | engineering | OK |

## Client Test Accounts — Google SSO (credo-group.co.uk)

Mirror of the Microsoft set. All should be in the `participants` table.

Shared password for all `credo-group.co.uk` accounts: `CdqXDZSKDVAr2qG`

| # | Email | Name | Role Title | Department | Group | Status |
|--:|-------|------|-----------|------------|-------|--------|
| 12 | `mark.director@credo-group.co.uk` | Mark Director | Director | Leadership | leadership | OK |
| 13 | `adam.finance@credo-group.co.uk` | Adam Finance | Finance Manager | Finance | finance | OK |
| 14 | `kevin.operations@credo-group.co.uk` | Kevin Operations | Operations Manager | Operations | operations | OK |
| 15 | `damon.operations@credo-group.co.uk` | Damon Operations | Operations Supervisor | Operations | operations | OK |
| 16 | `nicholas.field@credo-group.co.uk` | Nicholas Field | Field Engineer | Field Engineering | engineering | OK |
| 17 | `mathew.field@credo-group.co.uk` | Mathew Field | Field Engineer | Field Engineering | engineering | OK |

---

## Summary

| Category | Count | Status |
|----------|-------|--------|
| **Consultants (active)** | 3 | 2 in DB, 1 missing (Greg) |
| **Consultants (planned)** | 2 | Future — `@limitlessmodus.com` |
| **Client test accounts (Microsoft)** | 6 | All in `participants` |
| **Client test accounts (Google)** | 6 | All in `participants` |
| **Total unique people** | 8 | Fedor, Greg, + 6 test personas |

## Remaining Issues

1. ~~6 records in wrong table~~ — **Fixed 2026-03-06.** Moved mark.director, adam.finance, kevin.operations from `users` to `participants`.
2. **Greg not in DB** — `greg.kurnikov@odexpert.co.uk` needs to be added to the `users` table as a consultant.
3. **Seed engagement cleanup** — duplicate `fedor.vasilyev@tumai.cc` record in `users` tied to the "Nano Fibre (Pilot)" seed engagement (`a0000000-...`).

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
