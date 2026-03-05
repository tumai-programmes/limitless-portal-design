# Authentication & Authorisation

Design and planning documentation for the Limitless Portal authentication system.

## Strategy

The portal uses **SSO-only authentication** — no passwords stored or managed by the portal. Users sign in via their organisation's identity provider:

1. **Microsoft Entra ID** (Azure AD) — primary, implemented
2. **Google OAuth2** — planned (Phase 2)

Role and access are determined by the user record in the database, not by the login method. The SSO flow identifies the user by email; the backend looks up their role and engagement membership.

## Documents

| File | Purpose |
|------|---------|
| [login-page.md](login-page.md) | Login page layout spec — two SSO buttons, error states, loading states |
| [microsoft-sso.md](microsoft-sso.md) | Microsoft Entra ID integration — config, test accounts, callback flow |
| [google-sso.md](google-sso.md) | Google OAuth2 integration plan (future) |
| [role-model.md](role-model.md) | User roles, lookup logic, access control |
| [decisions-log.md](decisions-log.md) | Auth-related design decisions |

## Auth Flow

```
User visits portal
  → Login page shows two SSO buttons (Microsoft / Google)
  → User clicks their provider
  → Redirect to identity provider (Entra ID / Google)
  → User authenticates with their corporate or personal account
  → Redirect back with OAuth2 authorization code
  → Go backend exchanges code for tokens
  → Backend parses email from ID token
  → Looks up email in participants table → if found, route to participant dashboard
  → Looks up email in users table → if found, route to consultant dashboard
  → If not found → access denied (user must be pre-registered)
  → JWT issued for session management
```

## Key Principles

- **No passwords** — the portal never stores or verifies passwords
- **Pre-registration required** — users must exist in the database before they can log in (invited by consultant or admin)
- **Role from database** — the login method does not determine the role; the database record does
- **Provider-agnostic** — the backend handles multiple OAuth2 providers with a unified callback pattern
- **Session via JWT** — after SSO, the portal uses JWTs for stateless session management

---

*Part of the [Portal Design](../README.md) — [Limitless Modus](https://github.com/tumai-programmes/limitless) ecosystem.*
