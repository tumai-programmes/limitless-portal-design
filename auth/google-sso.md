# Google OAuth2 SSO

Google SSO is the second authentication method planned for the Limitless Portal, supporting clients whose organisations use Google Workspace rather than Microsoft 365.

## Status: Planned (Phase 2)

Not yet implemented. The Microsoft SSO flow is the template — Google follows the same OAuth2 authorization code pattern.

## Test Accounts Plan

Google Workspace domain for testing: **credo-group.co.uk** (Fedor's Google Workspace).

Fedor's primary Google identity: `fedor.vasilyev@tumai.com`

Test accounts to create in credo-group.co.uk (mirroring tumai.cc Microsoft accounts):

| # | Account | Role mapping | Department |
|-:|---------|-------------|-----------|
| 1 | `fedor.vasilyev@tumai.com` | Consultant | System |
| 2 | `mark.director@credo-group.co.uk` | Director | Leadership |
| 3 | `adam.finance@credo-group.co.uk` | Manager | Finance |
| 4 | `kevin.operations@credo-group.co.uk` | Manager | Operations |
| 5 | `damon.operations@credo-group.co.uk` | Participant (Operations Supervisor) | Operations |
| 6 | `nicholas.field@credo-group.co.uk` | Participant (Field Engineer) | Field Engineering |
| 7 | `mathew.field@credo-group.co.uk` | Participant (Field Engineer) | Field Engineering |

After creating the accounts, matching records must be inserted into Supabase (same pattern as the tumai.cc accounts in [microsoft-sso.md](microsoft-sso.md)).

## Concrete Next Steps

1. **Fedor** — create the Google Cloud OAuth2 app:
   - Go to [console.cloud.google.com](https://console.cloud.google.com)
   - Create project or reuse existing Tumai project
   - APIs & Services → OAuth consent screen → External → app name "Limitless Modus"
   - APIs & Services → Credentials → Create Credentials → OAuth Client ID → Web application
   - Set Authorized redirect URI: `https://nano.limitlessmodus.com/auth/google/callback`
   - Copy **Client ID** and **Client Secret**

2. **Fedor** — create test accounts in credo-group.co.uk (see table above)

3. **AI** — implement backend handlers, routes, config (mirrors Microsoft flow)

4. **AI** — seed matching records in Supabase for Google test accounts

5. **AI** — enable the Google button in LoginPage.vue (currently disabled with "Coming soon")

6. **Deploy + test** end-to-end with `fedor.vasilyev@tumai.com`

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

## Priority

Google SSO will be implemented when a client requires it. The Microsoft flow is sufficient for the first engagement (Nano Fibre).

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
