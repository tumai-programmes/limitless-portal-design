# Test Accounts — Complete Inventory

All test accounts registered in the Limitless Portal database (Supabase), mapped to SSO providers.

**Engagement:** Nano Fibre UK (`72ed7515-f133-469e-9e4e-ab2e5d6c9171`)

> A second seed engagement "Nano Fibre (Pilot)" (`a0000000-...`) exists with one duplicate `fedor.vasilyev@tumai.cc` consultant record — legacy from initial scaffolding.

## Consultant / Admin Accounts

These are in the `users` table and land on the **consultant dashboard** after login.

| # | Email | Name | Role | SSO Provider | Domain | Last Login |
|--:|-------|------|------|-------------|--------|------------|
| 1 | `fedor.vasilyev@tumai.cc` | Fedor Vasilyev | consultant | Microsoft | tumai.cc | 2026-03-05 |
| 2 | `fedor.vasilyev@tumai.com` | Fedor Vasilyev | consultant | Google | tumai.com | 2026-03-06 |

## Microsoft SSO Test Accounts (tumai.cc)

### In `users` table (land on consultant/internal dashboard)

| # | Email | Name | Role | Password | Department |
|--:|-------|------|------|----------|------------|
| 3 | `mark.director@tumai.cc` | Mark Director | director | Kaxa172305 | Leadership |
| 4 | `adam.finance@tumai.cc` | Adam Finance | manager | Joto995535 | Finance |
| 5 | `kevin.operations@tumai.cc` | Kevin Operations | manager | Bado798597 | Operations |

### In `participants` table (land on participant dashboard)

| # | Email | Name | Role Title | Password | Department | Group |
|--:|-------|------|-----------|----------|------------|-------|
| 6 | `damon.operations@tumai.cc` | Damon Operations | Operations Supervisor | Xaqo596036 | Operations | operations |
| 7 | `nicholas.field@tumai.cc` | Nicholas Field | Field Engineer | Xobo878227 | Field Engineering | engineering |
| 8 | `mathew.field@tumai.cc` | Mathew Field | Field Engineer | Mucu340428 | Field Engineering | engineering |

## Google SSO Test Accounts (credo-group.co.uk)

### In `users` table (land on consultant/internal dashboard)

| # | Email | Name | Role | Department |
|--:|-------|------|------|------------|
| 9 | `mark.director@credo-group.co.uk` | Mark Director | director | Leadership |
| 10 | `adam.finance@credo-group.co.uk` | Adam Finance | manager | Finance |
| 11 | `kevin.operations@credo-group.co.uk` | Kevin Operations | manager | Operations |

### In `participants` table (land on participant dashboard)

| # | Email | Name | Role Title | Department | Group |
|--:|-------|------|-----------|------------|-------|
| 12 | `damon.operations@credo-group.co.uk` | Damon Operations | Operations Supervisor | Operations | operations |
| 13 | `nicholas.field@credo-group.co.uk` | Nicholas Field | Field Engineer | Field Engineering | engineering |
| 14 | `mathew.field@credo-group.co.uk` | Mathew Field | Field Engineer | Field Engineering | engineering |

## Summary

| Metric | Count |
|--------|-------|
| **Total DB records** | 15 (9 users + 6 participants) |
| **Unique people** | 7 (Fedor + 6 test personas) |
| **Microsoft SSO (tumai.cc)** | 7 accounts (1 consultant + 3 users + 3 participants) |
| **Google SSO (credo-group.co.uk)** | 6 accounts (3 users + 3 participants) |
| **Google SSO (tumai.com)** | 1 account (Fedor — consultant) |
| **Engagements** | 2 (Nano Fibre UK + Nano Fibre Pilot seed) |

## How Roles Work

The SSO callback (`completeSSO` in `auth.go`) checks tables in this order:

1. **`participants`** table first — if found, user lands on the **participant dashboard** with their `role_title`
2. **`users`** table second — if found, user lands on the **consultant dashboard** with their `role` (consultant, director, manager, engineer)
3. **Neither** — user sees "not registered" error on the login page

## Notes

- Passwords are only listed for Microsoft accounts (Azure AD requires them). Google accounts use the user's existing Google password.
- The `credo-group.co.uk` Google Workspace passwords are managed in Google Admin and not stored here.
- The `fedor.vasilyev@tumai.cc` account appears twice in the `users` table — once for the real engagement and once for the Pilot seed engagement. The SSO flow picks the first match.

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
