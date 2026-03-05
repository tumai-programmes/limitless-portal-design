# Auth — Decisions Log

| # | Decision | Choice | Rationale | Date |
|---|----------|--------|-----------|------|
| A1 | Authentication method | SSO-only (no passwords) | Portal users are enterprise employees with existing identity providers. Storing passwords adds security liability and UX friction. | 2026-03-02 |
| A2 | Primary SSO provider | Microsoft Entra ID (Azure AD) | First client (Nano Fibre) uses Microsoft 365. Azure AD is the dominant enterprise identity provider in the UK. | 2026-03-02 |
| A3 | Secondary SSO provider | Google OAuth2 | Some clients use Google Workspace. Google OAuth2 follows the same authorization code pattern as Azure, minimal additional effort. | 2026-03-05 |
| A4 | Role determination | Database lookup by email | Role is stored in the database, not chosen at login time. This prevents users from escalating their own access and simplifies the login UI to a single flow. | 2026-03-05 |
| A5 | Dev-mode auth bypass | Remove | Moving to real SSO for all environments (including development). Test accounts created in Azure AD (tumai.cc) for development testing. No need for a separate dev-mode bypass path. | 2026-03-05 |
| A6 | Login page layout | Two SSO buttons (Microsoft + Google) | Clean, minimal login experience. Microsoft is active, Google is disabled with "coming soon" until implemented. No email/password fields. | 2026-03-05 |
| A7 | User pre-registration | Required | Users must exist in the database before they can log in. Prevents unauthorized access. Users are invited by the consultant when setting up an engagement. | 2026-03-05 |
| A8 | Session management | JWT (access + refresh tokens) | Stateless session management. Access token expires in 24h, refresh token in 30 days. Tokens stored in localStorage. | 2026-03-02 |
| A9 | Token delivery | URL query parameters on redirect | After SSO callback, the backend redirects to the SPA with tokens in the URL. The Vue router guard strips them immediately and stores in localStorage. Simple, works with SPA architecture. | 2026-03-02 |

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
