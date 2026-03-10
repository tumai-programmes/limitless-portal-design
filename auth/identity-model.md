# Two-Table Identity Model

How the Limitless Portal separates internal staff from client participants using two database tables, and how the SSO flow navigates between them.

## Rationale

The portal serves two fundamentally different user populations:

| Population | Relationship to Portal | Engagement Scope | Example |
|-----------|----------------------|------------------|---------|
| **Internal staff** | Operate the platform, manage multiple engagements | System-wide | Greg (consultant), Fedor (admin) |
| **Client participants** | Complete audits for a specific engagement | Per-engagement | Paul Director, Nick Operations |

These populations have different identity semantics:

- A **consultant** has one account that spans all engagements. They need a system-wide view.
- A **participant** is scoped to an engagement. The same person (email) can participate in multiple engagements, each time with a different role, department, and instrument assignment.

A single-table model would force one of two bad compromises: either participants lose per-engagement metadata, or consultants get duplicated across every engagement. The two-table model avoids both.

## The Two Tables

### `users` — Internal Portal Accounts

Reserved for consultants and admins who operate the platform.

| Column | Purpose |
|--------|---------|
| `id` | UUID, primary key |
| `email` | Unique, used for SSO lookup |
| `name` | Display name |
| `role` | `consultant` or `admin` |
| `engagement_id` | Optional default engagement context |

**One row per email. System-wide scope.**

### `participants` — Client Participant Records

One row per person per engagement. Rich metadata for audit routing.

| Column | Purpose |
|--------|---------|
| `id` | UUID, primary key |
| `email` | Not unique — same email can appear in multiple rows |
| `name` | Display name |
| `role_title` | Job title (Director, Operations Manager, Field Engineer) |
| `department` | Organisational unit |
| `participant_group` | Grouping within the engagement |
| `instruments_assigned` | Which audit instruments this person completes |
| `engagement_id` | FK to the specific engagement |

**One row per email per engagement. Engagement-scoped.**

## SSO Lookup Flow

When a user authenticates via SSO (Microsoft Entra ID or Google), the backend resolves their identity in a strict order:

```
Email from SSO provider
  │
  ▼
┌─────────────────────────────┐
│ 1. Check `users` table      │
│    GetUserByEmail(email)    │
└──────────┬──────────────────┘
           │
     ┌─────┴─────┐
     │  Found?   │
     └─────┬─────┘
       Yes │           No
           ▼            ▼
  Issue JWT with    ┌──────────────────────────┐
  users.id          │ 2. Check `participants`  │
  Route by role:    │    GetParticipantsByEmail │
  • consultant →    └──────────┬───────────────┘
    /dashboard                 │
  • participant →        ┌─────┴─────┐
    /participant-        │  Count?   │
    dashboard            └─────┬─────┘
                          0    │  1     │  2+
                          ▼    ▼        ▼
                       Access  Issue    Issue temp
                       denied  JWT     token →
                               with    /select-
                               p.id    engagement
                               →       (picker)
                               /participant-
                               dashboard
```

**Critical rule: `users` is checked first. If the email exists there, the `participants` table is never consulted and the engagement picker is never shown.**

This means:
- Consultants and admins are resolved immediately from `users`.
- Participant-only emails fall through to the `participants` path.
- Multi-engagement participants (2+ rows in `participants`) see an engagement picker.

## Multi-Engagement Participants

When a participant email has rows in multiple engagements:

1. The SSO callback finds 2+ `participants` rows.
2. A temporary token is issued with `role: "pending_selection"` (10-minute expiry).
3. The user is redirected to `/select-engagement` with this temp token.
4. The picker page calls `/auth/my-engagements` to list available engagements.
5. The user selects one. The backend issues a full JWT scoped to that engagement.
6. The user is redirected to `/participant-dashboard` with the engagement-scoped JWT.

This flow only works when the email exists exclusively in the `participants` table.

## JWT Claims

After identity resolution, the JWT contains:

| Claim | Source | Meaning |
|-------|--------|---------|
| `email` | SSO id_token | User's email address |
| `role` | Derived | `consultant`, `admin`, or `participant` |
| `user_id` | `users.id` or `participants.id` | The resolved identity row |
| `engagement_id` | From the resolved row | Active engagement context |

The `user_id` claim is particularly important: it determines which table the ID belongs to, which in turn determines how downstream queries work. The Participant Dashboard queries `submissions WHERE participant_id = $1` using this claim.

## Design Rules

1. **Participants must only exist in the `participants` table.** If a participant email also has a row in `users`, the SSO flow will resolve via `users` first, bypassing the engagement picker and potentially using the wrong ID type for downstream queries.

2. **The `users` table is for consultants and admins only.** The `role` column should only contain `consultant` or `admin`. Any other value is normalized to `participant` by `normalizeAuthRole()` as a safety net, but this indicates a data issue.

3. **Participant job titles are not auth roles.** "Director", "Manager", and "Engineer" are `role_title` values in the `participants` table. They are not auth roles. The auth role for all participants is `participant`.

4. **One `participants` row per engagement.** The same email can appear multiple times, but each row must reference a different `engagement_id`.

## Known Pitfall: Cross-Table Contamination

During initial development, some participant accounts were inserted into both `users` and `participants`. This caused:

- **Engagement picker bypass**: The SSO flow found the email in `users` first and issued a JWT without consulting `participants`. Multi-engagement users were locked to one engagement with no picker.
- **ID mismatch**: The JWT contained a `users.id` (e.g., `c1000000-...`), but submissions were linked via `participant_id` (e.g., `b1000000-...`). The Participant Dashboard returned empty results.
- **Role confusion**: The `users.role` column contained job titles (`director`, `manager`, `engineer`) instead of auth roles (`participant`). The frontend router treated these as consultant-tier roles.

### Resolution (2026-03-08)

1. **Database migration** (`016_normalize_user_roles.up.sql`): Updated the `users.role` check constraint to explicitly allow only `consultant`, `admin`, and `participant`. Normalized existing job-title roles to `participant`.
2. **Backend hardening** (`auth.go`): Added `normalizeAuthRole()` to treat any non-consultant/admin role as `participant` before JWT generation.
3. **Frontend hardening** (`router/index.ts`): Updated routing guards to use an `isConsultant` flag rather than checking for exact role strings.
4. **Data cleanup**: Removed 3 contaminated accounts (`paul.director`, `nick.operations`, `simon.field` `@credo-group.co.uk`) from the `users` table. NULLed their orphaned `submissions.user_id` FK references. These accounts now resolve correctly through the `participants`-only path with engagement picker support.

## Relationship to Other Docs

| Document | Covers |
|----------|--------|
| [role-model.md](role-model.md) | Access control matrix, what each role can do |
| [test-accounts.md](test-accounts.md) | Inventory of test accounts and their table placement |
| [test-accounts-assignment.md](test-accounts-assignment.md) | Which test accounts map to which engagements |
| [microsoft-sso.md](microsoft-sso.md) | Microsoft Entra ID configuration |
| [google-sso.md](google-sso.md) | Google OAuth2 configuration |

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
