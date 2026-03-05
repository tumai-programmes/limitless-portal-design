# Microsoft Entra ID (Azure AD) SSO

Microsoft SSO is the primary authentication method for the Limitless Portal. The first client (Nano Fibre) uses Microsoft corporate accounts.

## Status: Implemented

The full OAuth2/OIDC flow is implemented end-to-end:
- Frontend redirects to Azure login via `GET /auth/login`
- Azure redirects back to `GET /auth/callback` with authorization code
- Backend exchanges code for tokens, parses email from `id_token`
- Looks up user/participant in database, issues JWT, redirects to appropriate dashboard

## Azure App Registration

| Setting | Value |
|---------|-------|
| **Tenant ID** | `AZURE_TENANT_ID` env var (tumai.cc directory) |
| **Client ID** | `AZURE_CLIENT_ID` env var |
| **Client Secret** | `AZURE_CLIENT_SECRET` env var |
| **Redirect URI** | `https://nano.limitlessmodus.com/auth/callback` |
| **Scopes** | `openid email profile` |
| **Response type** | `code` (authorization code flow) |
| **Response mode** | `query` |

### Endpoints

| Endpoint | URL |
|----------|-----|
| Authorize | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize` |
| Token | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token` |

## Test Accounts (tumai.cc)

Test accounts created under the tumai.cc Azure AD tenant for development and testing.

| # | Account | Password | First Name | Last Name | Department | Portal Role |
|--:|---------|----------|------------|-----------|------------|-------------|
| 1 | mark.director@tumai.cc | Kaxa172305 | Mark | Director | Leadership | Director |
| 2 | adam.finance@tumai.cc | Joto995535 | Adam | Finance | Finance | Manager |
| 3 | kevin.operations@tumai.cc | Bado798597 | Kevin | Operations | Operations | Manager |
| 4 | damon.operations@tumai.cc | Xaqo596036 | Damon | Operations | Operations | Manager |
| 5 | nicholas.field@tumai.cc | Xobo878227 | Nicholas | Engineering | Field Engineering | Engineer |
| 6 | mathew.field@tumai.cc | Mucu340428 | Mathew | Engineering | Field Engineering | Engineer |

### Role Mapping Rationale

- **mark.director** → Director role: represents the client's managing director who submits Company Audit (01)
- **adam.finance** → Manager role: functional manager for Finance, submits Manager Audit (02)
- **kevin.operations, damon.operations** → Manager role: functional managers for Operations, submit Manager Audit (02)
- **nicholas.field, mathew.field** → Engineer role: field engineers, submit Engineer Mini-Audit (03)

### Testing Checklist

- [ ] Sign in with mark.director@tumai.cc → should land on participant dashboard with Director permissions
- [ ] Sign in with adam.finance@tumai.cc → should land on participant dashboard with Manager permissions
- [ ] Sign in with kevin.operations@tumai.cc → should land on participant dashboard
- [ ] Sign in with nicholas.field@tumai.cc → should land on participant dashboard with Engineer survey
- [ ] Sign in with fedor.vasilyev@tumai.cc → should land on consultant dashboard (existing admin user)
- [ ] Sign in with unregistered email → should see "not registered" error on login page

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `AZURE_CLIENT_ID` | Yes | Microsoft Entra ID application (client) ID |
| `AZURE_CLIENT_SECRET` | Yes | Client secret for the app registration |
| `AZURE_TENANT_ID` | Yes | Azure AD tenant ID (tumai.cc directory) |
| `AZURE_REDIRECT_URI` | Yes | OAuth2 callback URL (default: `https://nano.limitlessmodus.com/auth/callback`) |

## Backend Implementation

| File | Purpose |
|------|---------|
| `internal/handlers/auth.go` | `LoginRedirect` (redirect to Azure) + `Callback` (exchange code, issue JWT) |
| `internal/config/config.go` | Azure config fields + `AzureAuthorizeURL()` / `AzureTokenURL()` helpers |
| `internal/router/router.go` | Route registration: `GET /auth/login`, `GET /auth/callback` |

## Security Notes

- The `id_token` JWT from Azure is decoded (base64) but **not cryptographically verified** against Azure's signing keys. This is acceptable for Phase 1 because the token comes directly from Azure over HTTPS in the same HTTP response as the code exchange. For production hardening, signature verification should be added.
- The `code` exchange happens server-side (Go backend), so the client secret is never exposed to the browser.
- Tokens are passed to the frontend via URL query parameters on redirect, then stripped from the URL immediately by the Vue router guard.

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*
