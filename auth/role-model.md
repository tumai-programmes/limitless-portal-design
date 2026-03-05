# Role Model & Access Control

How user roles work in the Limitless Portal — who can do what, and how the system determines it.

## Key Principle

**Role is determined by the database, not the login method.**

When a user signs in via SSO (Microsoft or Google), the backend:
1. Extracts the email address from the identity provider's `id_token`
2. Looks up the email in the `participants` table first
3. If found → issues JWT with `role: "participant"` and the participant's specific engagement role
4. If not found → looks up in the `users` table (consultants, admins)
5. If found → issues JWT with the user's role from the database
6. If not found in either table → access denied

There is no "Sign In as Consultant" vs "Sign In as Participant" choice. The system knows who you are from your email.

## Roles

| Role | Database Table | Who | Portal Access |
|------|---------------|-----|---------------|
| **Consultant** | `users` | Greg Kurnikov | Full access — manage engagements, view all submissions, run interviews, trigger synthesis |
| **Admin** | `users` | Fedor (system) | System configuration, user management, engagement setup |
| **Director** | `participants` | Client MD/Director | Submit Company Audit (01), view engagement progress |
| **Manager** | `participants` | Functional managers | Submit Manager Audit (02), view own submission status |
| **Engineer** | `participants` | Field engineers | Submit Engineer Mini-Audit (03), minimal UI |

## Database Lookup Order

```
Email from SSO
  │
  ├─ participants table (engagement-scoped)
  │    ├─ Found → role = "participant", sub-role from participants.role
  │    │           engagement_id from participants.engagement_id
  │    │           → redirect to /participant-dashboard
  │    │
  │    └─ Not found → continue lookup
  │
  ├─ users table (system-wide)
  │    ├─ Found → role from users.role (consultant / admin)
  │    │           engagement_id from users.engagement_id
  │    │           → redirect to /dashboard
  │    │
  │    └─ Not found → access denied
  │
  └─ → redirect to /login?error=not_registered
```

## JWT Claims

After successful lookup, the JWT contains:

| Claim | Source | Example |
|-------|--------|---------|
| `email` | SSO id_token | `mark.director@tumai.cc` |
| `role` | Database record | `participant`, `consultant`, `admin` |
| `user_id` | Database record ID | `uuid` |
| `engagement_id` | Database record | `uuid` |
| `exp` | Generated | Access: 24h, Refresh: 30d |

## Access Control Matrix

| Resource | Consultant | Admin | Director | Manager | Engineer |
|----------|-----------|-------|----------|---------|----------|
| View all engagements | Yes | Yes | No | No | No |
| Create engagement | No | Yes | No | No | No |
| View engagement users | Yes | Yes | No | No | No |
| Invite users | Yes | Yes | No | No | No |
| View all submissions | Yes | Yes | No | No | No |
| Submit Company Audit (01) | No | No | Yes | No | No |
| Submit Manager Audit (02) | No | No | No | Yes | No |
| Submit Engineer Mini-Audit (03) | No | No | No | No | Yes |
| View own submission | No | No | Yes | Yes | Yes |
| Manage interviews | Yes | No | No | No | No |
| Trigger AI synthesis | Yes | No | No | No | No |
| View findings/reports | Yes | Yes | Yes | No | No |

## Pre-Registration Requirement

Users must be created in the database before they can log in:

1. **Consultant/Admin** — created in `users` table (seed data or admin UI)
2. **Participants** — created in `participants` table when the consultant sets up an engagement and invites client users

An unregistered email that successfully authenticates via SSO will see an error page explaining they need to be invited.

## Future: Auto-Registration

A future enhancement could allow auto-registration when a user signs in from a recognised domain (e.g., `@nanofibre.co.uk`). The user would be created with a default role and assigned to the engagement associated with that domain. This is not planned for Phase 1.

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
