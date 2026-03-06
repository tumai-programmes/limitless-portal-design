# Google OAuth2 SSO

Google SSO is the second authentication method for the Limitless Portal, supporting clients whose organisations use Google Workspace rather than Microsoft 365.

## Status: Implemented (2026-03-06)

Fully implemented and deployed alongside Microsoft SSO. Both providers use the same shared `completeSSO` logic for role lookup and JWT issuance.

## Google Cloud Project

| Property | Value |
|----------|-------|
| Project | Limitless Modus (dedicated GCP project) |
| OAuth App | Limitless Portal (Web application) |
| Client ID | `760513599819-h3rdackqi02sfnhqfil1lvnobtg5qo42.apps.googleusercontent.com` |
| Redirect URI | `https://nano.limitlessmodus.com/auth/google/callback` |
| Consent screen | External, app name "Limitless Modus" |
| User support email | `fedor.vasilyev@tumai.com` |
| Branded email | `hello@limitlessmodus.com` (Google Workspace, secondary domain) |

## Test Accounts

Google Workspace domain for testing: **credo-group.co.uk** (Fedor's Google Workspace).

Fedor's primary Google identity: `fedor.vasilyev@tumai.com`

| # | Account | Role mapping | Department | DB Table |
|-:|---------|-------------|-----------|----------|
| 1 | `fedor.vasilyev@tumai.com` | Consultant | System | `users` |
| 2 | `mark.director@credo-group.co.uk` | Director | Leadership | `users` |
| 3 | `adam.finance@credo-group.co.uk` | Manager | Finance | `users` |
| 4 | `kevin.operations@credo-group.co.uk` | Manager | Operations | `users` |
| 5 | `damon.operations@credo-group.co.uk` | Participant (Operations Supervisor) | Operations | `participants` |
| 6 | `nicholas.field@credo-group.co.uk` | Participant (Field Engineer) | Field Engineering | `participants` |
| 7 | `mathew.field@credo-group.co.uk` | Participant (Field Engineer) | Field Engineering | `participants` |

All accounts seeded in Supabase (Nano Fibre UK engagement: `72ed7515-f133-469e-9e4e-ab2e5d6c9171`).

## Implementation Checklist

- [x] Google Cloud project created (Limitless Modus)
- [x] OAuth consent screen configured (External)
- [x] OAuth Client ID and Secret generated
- [x] `limitlessmodus.com` added as secondary domain in Google Workspace
- [x] `hello@limitlessmodus.com` email account created
- [x] DNS records configured (MX, SPF, DMARC) for `limitlessmodus.com`
- [x] Backend: `config.go` — Google config fields added
- [x] Backend: `auth.go` — `GoogleLoginRedirect` and `GoogleCallback` handlers
- [x] Backend: `auth.go` — shared `completeSSO` method (refactored from Azure-only)
- [x] Backend: `router.go` — `/auth/google/login` and `/auth/google/callback` routes
- [x] Frontend: `LoginPage.vue` — Google button enabled (was disabled with "Coming soon")
- [x] Frontend: `auth.ts` — `loginWithGoogle()` function added
- [x] Server `.env` on `bcl-limapp-10` — Google env vars added
- [x] Supabase — Google test accounts seeded (4 users + 3 participants)
- [ ] End-to-end test with `fedor.vasilyev@tumai.com`

## Implementation Plan

### 1. Google Cloud Console Setup

- Create a project in Google Cloud Console (or use existing Tumai project)
- Enable the "Google Identity" / "OAuth consent screen" API
- Create OAuth2 credentials (Web application type)
- Set authorized redirect URI: `https://nano.limitlessmodus.com/auth/google/callback`
- Configure OAuth consent screen (external, limited to verified domains)

### 2. Backend Changes

Add to `internal/config/config.go`:

```go
GoogleClientID     string
GoogleClientSecret string
GoogleRedirectURI  string
```

Add two new handlers in `internal/handlers/auth.go`:

- `GoogleLoginRedirect` — redirect to `https://accounts.google.com/o/oauth2/v2/auth`
- `GoogleCallback` — exchange code at `https://oauth2.googleapis.com/token`, parse `id_token` for email

Add routes in `internal/router/router.go`:

```
GET /auth/google/login    → GoogleLoginRedirect
GET /auth/google/callback → GoogleCallback
```

### 3. Frontend Changes

Add a "Sign in with Google" button to `LoginPage.vue` that calls `window.location.href = '/auth/google/login'`.

### 4. Environment Variables

| Variable | Description |
|----------|-------------|
| `GOOGLE_CLIENT_ID` | Google OAuth2 client ID |
| `GOOGLE_CLIENT_SECRET` | Google OAuth2 client secret |
| `GOOGLE_REDIRECT_URI` | Callback URL (default: `https://nano.limitlessmodus.com/auth/google/callback`) |

### 5. Google OAuth2 Endpoints

| Endpoint | URL |
|----------|-----|
| Authorize | `https://accounts.google.com/o/oauth2/v2/auth` |
| Token | `https://oauth2.googleapis.com/token` |
| Scopes | `openid email profile` |

## Architecture Pattern

The Google flow mirrors the Microsoft flow exactly:

```
Browser → /auth/google/login → Google OAuth consent screen
  → /auth/google/callback?code=...
  → Exchange code for tokens (server-side)
  → Parse email from id_token
  → Look up user/participant in database
  → Issue JWT → redirect to dashboard
```

The backend callback handler uses the same `parseIDTokenClaims()` function — Google and Microsoft both return standard OIDC `id_token` JWTs with `email` and `name` claims.

## Multi-Tenant Note

The current redirect URI is bound to `nano.limitlessmodus.com`. When moving to a multi-tenant portal, additional redirect URIs can be added in the Google Cloud Console (e.g., `https://app.limitlessmodus.com/auth/google/callback`). The backend `GOOGLE_REDIRECT_URI` env var would be updated accordingly.

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
