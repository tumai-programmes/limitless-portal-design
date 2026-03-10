# Portal — Invincibility Blueprint Digital Platform

Architecture specification for the Limitless Modus portal — an authenticated, data-intensive web application that replaces Greg Kurnikov's Excel/Word-based methodology delivery with a digital diagnostic platform.

**Code repository:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal)
**Live URL:** [portal.limitlessmodus.com](https://portal.limitlessmodus.com/)
**Dev URL:** [dev-portal.limitlessmodus.com](https://dev-portal.limitlessmodus.com/)
**Design repo:** [tumai-programmes/limitless-portal-design](https://github.com/tumai-programmes/limitless-portal-design)
**SQL operations:** [tumai-hq/sql-hub/limitless-portal](https://github.com/tumai-hq/sql-hub)

*Last updated: 2026-03-09*

---

## Table of Contents

- [1. Architecture Overview](#1-architecture-overview)
  - [1.1 Technology Stack](#11-technology-stack)
  - [1.2 System Context](#12-system-context)
- [2. Database](#2-database)
  - [2.1 Strategy and Provider](#21-strategy-and-provider)
  - [2.2 Supabase Tier Analysis — Production Readiness](#22-supabase-tier-analysis--production-readiness)
  - [2.3 Schema Overview](#23-schema-overview)
  - [2.4 Migrations](#24-migrations)
  - [2.5 Row-Level Security](#25-row-level-security)
  - [2.6 Connection Management](#26-connection-management)
  - [2.7 Backup and Recovery](#27-backup-and-recovery)
  - [2.8 Migration Runner](#28-migration-runner)
- [3. File Storage](#3-file-storage)
- [4. Authentication](#4-authentication)
  - [4.1 SSO Providers](#41-sso-providers)
  - [4.2 Auth Flow](#42-auth-flow)
  - [4.3 Dev-Mode Bypass](#43-dev-mode-bypass)
- [5. User Roles and Access](#5-user-roles-and-access)
- [6. Data Model](#6-data-model)
  - [6.1 Core Entities](#61-core-entities)
  - [6.2 Instrument Types](#62-instrument-types)
- [7. API Endpoints](#7-api-endpoints)
  - [7.1 Public Endpoints](#71-public-endpoints)
  - [7.2 Authentication](#72-authentication)
  - [7.3 Engagements](#73-engagements)
  - [7.4 Participants (Admin)](#74-participants-admin)
  - [7.5 Submissions](#75-submissions)
  - [7.6 Participant Flow](#76-participant-flow)
  - [7.7 Uploads (S3-backed)](#77-uploads-s3-backed)
  - [7.8 Communications](#78-communications)
  - [7.9 Email Tracking](#79-email-tracking)
  - [7.10 Admin](#710-admin)
- [8. Frontend](#8-frontend)
  - [8.1 Pages and Routes](#81-pages-and-routes)
  - [8.2 Audit Wizards](#82-audit-wizards)
  - [8.3 Question Components](#83-question-components)
  - [8.4 Wizard Layout Components](#84-wizard-layout-components)
- [9. Email Notifications](#9-email-notifications)
- [10. Deployment](#10-deployment)
  - [10.1 Infrastructure](#101-infrastructure)
  - [10.2 CI/CD Pipeline](#102-cicd-pipeline)
  - [10.3 Deployment Files](#103-deployment-files)
- [11. Implementation Status](#11-implementation-status)
  - [11.1 What Is Built](#111-what-is-built)
  - [11.2 What Is Scaffolded / Partial](#112-what-is-scaffolded--partial)
  - [11.3 What Is Not Started](#113-what-is-not-started)
- [12. Implementation Phases](#12-implementation-phases)
- [13. Design Decisions Log](#13-design-decisions-log)
- [14. Open Questions and Risks](#14-open-questions-and-risks)

---

## 1. Architecture Overview

### 1.1 Technology Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| **Frontend** | Vue 3 (Composition API) + TypeScript + Vite | SPA served as static files by Go backend |
| **UI Components** | DevExtreme (Vue edition) | Wizard/stepper, data grids, form editors, file upload |
| **Backend** | Go 1.24+ (gorilla/mux) | REST API; single-binary deployment |
| **Database** | Supabase (managed PostgreSQL) via `pgx` / `pgxpool` | Repository pattern; connection string is the only change between providers |
| **File Storage** | AWS S3 | Pre-signed URLs for direct browser upload; engagement-isolated prefixes |
| **Auth** | Microsoft Entra ID (Azure AD) + Google OAuth | SSO via OAuth2/OIDC |
| **Email** | AWS SES | Transactional emails (invitation, reminder, completion) with open/click tracking |
| **AI Synthesis** | Anthropic API (Claude) | Planned — transcript + audit data to diagnostic findings |
| **Deployment** | Docker + HAProxy + GitHub Actions | BCL self-hosted infrastructure |

### 1.2 System Context

```
                                    ┌─────────────────────────┐
                                    │     Cloudflare DNS      │
                                    └────────────┬────────────┘
                                                 │
                                    ┌────────────▼────────────┐
                                    │  bcl-haproxy-20 (SSL)   │
                                    └────────────┬────────────┘
                                                 │
                                    ┌────────────▼────────────┐
                                    │  portal.limitlessmodus.com│
                                    │  ┌──────────────────┐   │
                                    │  │   Go Backend      │   │
                                    │  │   :8891           │   │
                                    │  └──────────────────┘   │
                                    │  ┌──────────────────┐   │
                                    │  │  Vue SPA (static) │   │
                                    │  └──────────────────┘   │
                                    └─────────────────────────┘
                                                 │
              ┌──────────────┬───────────────────┼───────────────────┬──────────────┐
              │              │                   │                   │              │
    ┌─────────▼────────┐ ┌───▼───────────┐ ┌────▼──────┐ ┌─────────▼────────┐ ┌───▼──────────┐
    │ Supabase         │ │ AWS S3        │ │ Microsoft │ │ AWS SES          │ │ Anthropic    │
    │ (PostgreSQL)     │ │ (artifacts)   │ │ Entra ID  │ │ (email)          │ │ API (Claude) │
    │ Structured data  │ │ Raw uploads   │ │ + Google  │ │ (notifications)  │ │ (synthesis)  │
    └──────────────────┘ └───────────────┘ └───────────┘ └──────────────────┘ └──────────────┘
```

---

## 2. Database

### 2.1 Strategy and Provider

The Go backend uses a **repository pattern** (interface-based abstraction) so the database provider can be swapped without changing business logic. All queries use standard PostgreSQL via the `pgx` driver. The connection string is the only thing that changes between environments.

| Option | Type | Status | Use Case |
|--------|------|--------|----------|
| **Supabase** (managed PostgreSQL) | Cloud | **Active** — production | Default for all engagements; managed backups, dashboard for inspection |
| Local PostgreSQL | Self-hosted | *Reserve* | Development, offline testing |
| BCL PostgreSQL (`bcl-postgres-60`) | Self-hosted | *Reserve* | On-premises for UK-only data residency |
| Other cloud (Neon, AWS RDS) | Cloud | *Future* | Alternative if Supabase pricing or features become limiting |

A `DB_PROVIDER` environment variable exists in the config struct but is currently informational only — the backend connects to whatever `DATABASE_URL` points to.

### 2.2 Supabase Tier Analysis — Production Readiness

With a real client (Nano Fibre) being onboarded, the current Supabase Free tier needs evaluation against production requirements.

#### Free Tier Limitations (as of March 2026)

| Resource | Free Tier Limit | Current Usage | Production Estimate (1 engagement, ~30 users) | Risk |
|----------|----------------|---------------|-----------------------------------------------|------|
| **Database size** | 500 MB | ~5 MB (pilot seed + dev data) | ~50–200 MB (responses, drafts, logs, uploads metadata) | Low for 1 client; will hit limit at 3–5 concurrent engagements |
| **Database egress** | 5 GB / month | Minimal | ~2–10 GB (wizard auto-save, dashboard reads, API queries) | Medium — auto-save on every field change generates steady egress |
| **Direct connections** | 60 | 1 (dev) | 5–15 (Go backend pool + admin queries) | Low |
| **Pooler connections** | 200 | Not used (Go uses pgxpool) | Not applicable | N/A |
| **Daily backups** | 7-day retention | Active | Adequate for dev; **insufficient for production SLA** | **High** — no point-in-time recovery, no manual backup/restore API |
| **Branching** | Not available | — | Useful for staging | Low |
| **Read replicas** | Not available | — | Not needed now | Low |
| **SLA** | None | — | **No uptime guarantee** | **Critical** — production with real client needs uptime commitment |
| **Support** | Community only | — | **No priority support** | **High** — if DB goes down during engagement, no escalation path |
| **Pause after inactivity** | 7 days | Active | Would pause between engagement sessions | **Critical** — auto-pause would cause downtime for active client |

#### Recommendation: Upgrade to Supabase Pro

For production readiness with a real client, **Supabase Pro ($25/month)** is the minimum viable tier:

| Feature | Free | Pro ($25/mo) | Why It Matters |
|---------|------|-------------|----------------|
| Database size | 500 MB | 8 GB | Room for 10+ concurrent engagements |
| Egress | 5 GB/mo | 250 GB/mo | Auto-save + multiple users won't hit limit |
| Backups | 7-day daily | 14-day daily + point-in-time recovery | Can restore to any point if data corruption occurs |
| No auto-pause | No | **Yes** | Portal stays available between engagement sessions |
| SLA | None | 99.9% uptime | Contractual commitment for client |
| Support | Community | Email support | Escalation path for incidents |
| Log retention | 1 day | 7 days | Debug production issues |
| Branching | No | Yes | Safe schema changes via staging branch |

**Action items:**
1. Upgrade Supabase project to Pro tier before client onboarding
2. Verify the project is in EU (Frankfurt) region for UK data residency
3. Enable point-in-time recovery (PITR) once on Pro
4. Document the backup/restore procedure
5. Test database pause/resume behaviour doesn't affect current setup

#### Alternative: Self-hosted PostgreSQL

If Supabase costs become a concern at scale, the repository pattern allows migration to `bcl-postgres-60` (self-hosted) by changing `DATABASE_URL`. This requires:
- Manual backup/restore setup (pg_dump cron)
- Manual connection pooling (PgBouncer)
- Manual SSL certificate management
- No web dashboard (use DataGrip or psql)

Current recommendation: stay on Supabase for managed simplicity; revisit if running 5+ engagements or if total data exceeds Pro tier limits.

### 2.3 Schema Overview

The database has evolved through 10 migrations into the following structure:

```
engagements
  ├── id (UUID PK)
  ├── client_name, client_legal_name
  ├── status (draft | active | completed | archived)
  ├── slug (UNIQUE), allowed_email_domains (TEXT[]), settings (JSONB)
  ├── logo_url, logo_dark_url
  ├── due_date
  ├── created_at, updated_at
  │
  ├── participants (N per engagement)
  │   ├── id, engagement_id, name, email, role_title, department
  │   ├── participant_group, instruments_assigned (JSONB)
  │   ├── is_anonymous, notes, due_date_override
  │   └── created_at
  │
  ├── users (N per engagement — auth/login records)
  │   ├── id, engagement_id, email, name
  │   ├── role (consultant | admin | director | manager | engineer)
  │   ├── azure_oid, invited_at, last_login
  │   └── UNIQUE(engagement_id, email)
  │
  ├── submissions (N per engagement)
  │   ├── id, engagement_id, user_id, participant_id
  │   ├── instrument_type, status
  │   ├── consent_given, consent_timestamp
  │   ├── progress_percent, sections_completed
  │   ├── started_at, submitted_at, reviewed_at
  │   ├── first_opened_at, last_saved_at
  │   │
  │   ├── responses (N per submission)
  │   │   ├── id, submission_id, section, field_key
  │   │   ├── response_type, value_text, value_json
  │   │   └── updated_at
  │   │
  │   ├── section_drafts (N per submission — wizard auto-save)
  │   │   ├── id, submission_id, section_key
  │   │   ├── answers (JSONB), is_complete
  │   │   └── last_saved_at
  │   │
  │   └── support_questions (N per submission)
  │       ├── id, submission_id, section_key
  │       └── subject, message, created_at
  │
  ├── uploads (N per engagement)
  │   ├── id, engagement_id, submission_id, user_id
  │   ├── filename, mime_type, size_bytes, storage_path
  │   ├── section, field_key
  │   └── uploaded_at
  │
  ├── interviews (N per engagement)
  │   ├── id, engagement_id, interviewee_id, instrument_type
  │   ├── status, scheduled_at, completed_at
  │   └── transcript_text, transcript_source
  │
  ├── findings (N per engagement)
  │   ├── id, engagement_id, finding_type, title, description
  │   ├── toc_dimension, severity, evidence_refs, sources
  │   └── created_at
  │
  ├── notification_log (N per engagement)
  │   ├── id, engagement_id, participant_id
  │   ├── notification_type, instrument_type, channel
  │   ├── recipient_email, subject, status, error_message
  │   ├── sent_by, metadata (JSONB), tracking_token (UUID)
  │   └── created_at
  │
  └── email_events (N per notification)
      ├── id, tracking_token (FK → notification_log)
      ├── event_type (open | click | delivery | bounce)
      ├── ip_address, user_agent, url_clicked
      ├── metadata (JSONB)
      └── created_at

audit_log
  ├── id, user_id, engagement_id
  ├── action, entity_type, entity_id
  ├── details (JSONB)
  └── created_at
```

### 2.4 Migrations

All migrations are plain SQL files in `migrations/`, portable across PostgreSQL providers.

| Migration | Purpose |
|-----------|---------|
| `001_initial_schema` | Core tables: engagements, users, submissions, responses, uploads, interviews, findings, audit_log |
| `002_enable_rls` | Row-level security on initial tables; policies for `postgres` and `service_role` |
| `003_wizard_data_model` | Participants table, section_drafts, submission extensions (consent, progress), pilot seed data |
| `004_multi_tenant` | Engagement slug, allowed_email_domains, settings JSONB |
| `005_support_questions` | Support questions table (in-wizard "ask a question" feature) |
| `006_notification_log` | Notification log, engagement/participant due dates |
| `007_engagement_logo` | Engagement logo_url |
| `008_uploads_table` | Recreated uploads table with gen_random_uuid() and relaxed FK |
| `009_logo_dark_url` | Dark mode logo URL |
| `010_email_tracking` | Tracking token on notification_log, email_events table |

### 2.5 Row-Level Security

RLS is enabled on the initial 8 tables (migration 002) with policies for `postgres` and `service_role` roles. **Gap: tables added in migrations 003–010 do not have RLS policies:**

| Table | RLS Status |
|-------|------------|
| engagements, users, submissions, responses, uploads*, interviews, findings, audit_log | Enabled (migration 002) |
| participants, section_drafts | **No RLS** |
| support_questions | **No RLS** |
| notification_log, email_events | **No RLS** |

*uploads was dropped and recreated in migration 008 — RLS status depends on whether the recreation preserved the original policies.

**Risk assessment:** Since the Go backend connects as `postgres` role (which has full access through existing policies), the missing RLS does not currently block functionality. However, if Supabase client-side access or PostgREST is ever used, these tables would be unprotected. **Recommendation:** Add RLS to all newer tables before production to maintain defense-in-depth.

### 2.6 Connection Management

The backend uses `pgxpool.Pool` (Go-native connection pooling), not Supabase's built-in PgBouncer pooler. This is appropriate for a single Go backend connecting to Supabase. Configuration:

- Pool connects via `DATABASE_URL` (direct connection string)
- Default pgxpool settings (max connections determined by pgxpool defaults)
- No explicit pool size configuration in code

For production, consider setting explicit pool limits (`pgxpool.Config.MaxConns`) to avoid exhausting Supabase's connection limit.

### 2.7 Backup and Recovery

| Scenario | Current State | Production Requirement |
|----------|--------------|----------------------|
| **Daily backups** | Supabase Free: 7-day retention | Pro: 14-day + PITR |
| **Point-in-time recovery** | Not available on Free | Required — enables recovery from data corruption mid-engagement |
| **Manual export** | Can run `pg_dump` against direct connection | Document and test procedure |
| **Disaster recovery plan** | Not documented | Create runbook before first client goes live |

### 2.8 Migration Runner

Migrations are currently applied **manually** via `psql` or the Supabase SQL editor. There is no auto-migration on startup and no `make migrate` target that works.

**Production implication:** Each deployment that includes schema changes requires manual migration. This is acceptable for the current single-engagement phase but should be automated (e.g., `golang-migrate` or `goose`) before scaling.

---

## 3. File Storage

Raw artifacts (evidence uploads, transcripts, generated reports) are stored in AWS S3, separate from the database.

| Aspect | Detail |
|--------|--------|
| **Provider** | AWS S3 (eu-west-2, London region) |
| **Bucket structure** | `s3://limitless-portal/{engagement-id}/{instrument}/{filename}` |
| **Upload flow** | Browser → presigned URL (from Go backend) → direct upload to S3 → confirm metadata to DB |
| **Download** | Go backend generates presigned download URL → browser fetches from S3 |
| **Implementation** | `internal/services/s3.go` — presign, confirm, delete; `internal/handlers/uploads.go` — 4 endpoints |
| **Status** | **Implemented** — presign, confirm, list, delete endpoints all wired |

---

## 4. Authentication

### 4.1 SSO Providers

Two OAuth2/OIDC providers are implemented:

| Provider | Use Case | Endpoints |
|----------|---------|-----------|
| **Microsoft Entra ID** (Azure AD) | Primary — corporate clients (Nano Fibre uses Microsoft) | `/auth/login`, `/auth/callback` |
| **Google OAuth** | Secondary — for participants with Google Workspace accounts | `/auth/google/login`, `/auth/google/callback` |

### 4.2 Auth Flow

```
User visits portal.limitlessmodus.com
  → Login page shows "Sign in with Microsoft" and "Sign in with Google"
  → Redirected to chosen identity provider
  → Authenticates with corporate/personal account
  → Redirected back with OAuth2 authorization code
  → Go backend exchanges code for tokens, parses email from id_token
  → Looks up email in participants table (per engagement)
  → JWT issued for session management
  → Role and engagement determined from database record
  → Multi-engagement users see EngagementPickerPage
```

### 4.3 Dev-Mode Bypass

When `JWT_SECRET` starts with `dev-`, the login page shows dev-mode buttons that bypass SSO entirely. This allows local development without Azure/Google credentials configured.

---

## 5. User Roles and Access

| Role | Who | Portal Access |
|------|-----|---------------|
| **Consultant** | Greg Kurnikov | Full access — manage engagements, view all submissions, communications, trigger synthesis |
| **Admin** | Fedor (system) | System configuration, engagement setup, user/participant management |
| **Participant (Director)** | Client director/MD | Submit Company Audit (01), view own progress |
| **Participant (Manager)** | Client functional managers | Submit Manager Audit (02), view own progress |
| **Participant (Engineer)** | Client field engineers | Submit Engineer Mini-Audit (03), minimal UI |

The system has two related but distinct identity tables:
- **`users`** — login/auth records linked to Azure/Google OID
- **`participants`** — engagement-specific records with role, department, assigned instruments

A user authenticates via SSO, and the backend matches their email to participant records to determine engagement access and assigned instruments.

---

## 6. Data Model

### 6.1 Core Entities

See [2.3 Schema Overview](#23-schema-overview) for the full entity-relationship structure. Key relationships:

- An **Engagement** has many Participants, Users, Submissions, Uploads, Interviews, Findings, and NotificationLogs
- A **Participant** has many Submissions (one per assigned instrument)
- A **Submission** has many SectionDrafts (auto-save state per wizard section) and Responses (final saved field values)
- A **NotificationLog** entry has many EmailEvents (open/click tracking)

### 6.2 Instrument Types

| Code | Instrument | Submission Mode | Implementation Status |
|------|-----------|-----------------|----------------------|
| `company_audit` | Company Audit (01) | Multi-section wizard, auto-save, file uploads | **Implemented** |
| `manager_audit` | Manager Audit (02) | Multi-section wizard with department-conditional routing | **Implemented** |
| `engineer_mini_audit` | Engineer Mini-Audit (03) | Short survey wizard with job-type module selection | **Implemented** |
| `director_interview` | Director Interview (04) | Scheduled by consultant; transcript captured post-call | Not started |
| `manager_interview` | Manager Interview (05) | Scheduled by consultant; transcript captured post-call | Not started |
| `engineer_interview` | Engineer Interview (06) | Scheduled by consultant; transcript captured post-call | Not started |
| `operations_audit` | Operations Audit | Reserved in schema | Not started |
| `finance_audit` | Finance Audit | Reserved in schema | Not started |

---

## 7. API Endpoints

All endpoints return JSON. Protected routes require a JWT Bearer token or API key header.

### 7.1 Public Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/t/c/{token}` | Email click tracking (redirect) |
| GET | `/t/o/{token}.png` | Email open tracking (1x1 pixel) |

### 7.2 Authentication

| Method | Path | Description |
|--------|------|-------------|
| GET | `/auth/login` | Redirect to Microsoft Entra ID |
| GET | `/auth/callback` | Azure OAuth2 callback |
| GET | `/auth/google/login` | Redirect to Google OAuth |
| GET | `/auth/google/callback` | Google OAuth2 callback |
| GET | `/auth/me` | Current user + role + engagement |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Revoke session |
| GET | `/auth/my-engagements` | List engagements for current user |
| POST | `/auth/select-engagement` | Select active engagement (multi-engagement) |

### 7.3 Engagements

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements` | JWT | List all engagements |
| GET | `/api/engagements/{id}` | JWT | Get engagement details |
| POST | `/api/engagements` | Consultant | Create engagement |
| PUT | `/api/engagements/{id}` | Consultant | Update engagement |
| POST | `/api/engagements/{id}/archive` | Consultant | Archive engagement |

### 7.4 Participants (Admin)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements/{id}/participants` | Consultant | List participants |
| POST | `/api/engagements/{id}/participants` | Consultant | Create participant |
| POST | `/api/engagements/{id}/participants/import` | Consultant | Bulk import participants |
| PUT | `/api/participants/{id}` | Consultant | Update participant |
| DELETE | `/api/participants/{id}` | Consultant | Delete participant |

### 7.5 Submissions

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements/{id}/submissions` | JWT | List submissions for engagement |
| POST | `/api/engagements/{id}/submissions` | JWT | Create submission |
| GET | `/api/submissions/{id}` | JWT | Get submission detail |
| GET | `/api/submissions/{id}/responses` | JWT | Get all responses for submission |
| PUT | `/api/submissions/{id}/responses` | JWT | Save/update response (auto-save) |

### 7.6 Participant Flow

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/participant/dashboard` | JWT | Participant's assigned audits and progress |
| GET | `/api/participant/submissions/{id}` | JWT | Get submission with drafts |
| PUT | `/api/participant/submissions/{id}/sections` | JWT | Save section draft (auto-save) |
| POST | `/api/participant/submissions/{id}/consent` | JWT | Record consent acceptance |
| POST | `/api/participant/submissions/{id}/submit` | JWT | Final submission |
| PUT | `/api/participant/submissions/{id}/progress` | JWT | Update progress percent |
| POST | `/api/participant/submissions/{id}/questions` | JWT | Submit support question |

### 7.7 Uploads (S3-backed)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/uploads/presign` | JWT | Generate S3 presigned upload URL |
| POST | `/api/uploads/confirm` | JWT | Confirm upload, save metadata |
| GET | `/api/uploads` | JWT | List uploads (filtered by submission/section/field) |
| DELETE | `/api/uploads/{id}` | JWT | Delete from S3 + metadata |

### 7.8 Communications

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements/{id}/communications` | Consultant | Communications view with participant status |
| POST | `/api/engagements/{id}/pre-invite/{pid}` | Consultant | Send pre-invitation email |
| POST | `/api/engagements/{id}/invite/{pid}` | Consultant | Send full invitation email |
| POST | `/api/engagements/{id}/remind/{pid}` | Consultant | Send reminder email |
| POST | `/api/engagements/{id}/completion/{pid}` | Consultant | Send completion/thank-you email |
| GET | `/api/engagements/{id}/communications/{pid}/history` | Consultant | Per-participant communication history |

Legacy generic endpoints (will be deprecated):

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/notifications/invite` | Consultant | Generic invitation email |
| POST | `/api/notifications/remind` | Consultant | Generic reminder email |

### 7.9 Email Tracking

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/t/c/{token}` | Public | Click tracking — records event, redirects to destination URL |
| GET | `/t/o/{token}.png` | Public | Open tracking — records event, serves 1x1 transparent PNG |

### 7.10 Admin

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/admin/submissions/{id}/reset` | Consultant | Reset submission to clean state (danger zone) |

---

## 8. Frontend

### 8.1 Pages and Routes

| Route | Page | Layout | Role | Status |
|-------|------|--------|------|--------|
| `/login` | LoginPage | SingleCard | Guest | Implemented |
| `/dashboard` | DashboardPage | SideNavOuterToolbar | Consultant | Implemented |
| `/audits` | AuditsPage | SideNavOuterToolbar | Consultant | Implemented |
| `/communications` | CommunicationsPage | SideNavOuterToolbar | Consultant | Implemented |
| `/engagement-settings` | EngagementSettingsPage | SideNavOuterToolbar | Consultant | Implemented |
| `/interviews` | InterviewsPage | SideNavOuterToolbar | Consultant | Placeholder |
| `/findings` | FindingsPage | SideNavOuterToolbar | Consultant | Placeholder |
| `/select-engagement` | EngagementPickerPage | SingleCard | Any | Implemented |
| `/participant-dashboard` | ParticipantDashboard | SingleCard | Participant | Implemented |
| `/company-audit` | CompanyAuditWizard | EmptyLayout | Participant | Implemented |
| `/manager-audit` | ManagerAuditWizard | EmptyLayout | Participant | Implemented |
| `/engineer-mini-audit` | EngineerMiniAuditWizard | EmptyLayout | Participant | Implemented |

### 8.2 Audit Wizards

All three audit instruments share a common wizard architecture:
- **Instrument definitions** in `frontend/src/instruments/` (TypeScript objects defining sections, questions, types)
- **WizardLayout** with sidebar navigation, context banner, progress tracking
- **QuestionRenderer** maps question type to the appropriate component
- **Auto-save** via Pinia wizard store (debounced PUT to `/api/participant/submissions/{id}/sections`)
- **Consent step** at the start, **review/submit summary** at the end

### 8.3 Question Components

13 reusable question type components:

| Component | Type |
|-----------|------|
| QFreeText | Short text input |
| QFreeTextLong | Multi-line text area |
| QSingleSelect | Single choice (radio) |
| QMultiSelect | Multiple choice (checkboxes) |
| QMultiSelectCapped | Multi-select with maximum selection limit |
| QNumberInput | Numeric input |
| QRatingScale | Rating scale (e.g. 1–5 or custom labels) |
| QRatingScaleWithEvidence | Rating + free-text evidence field |
| QFileUpload | File upload (S3-backed) |
| QChecklistUpload | Checklist items with upload per item |
| QConsentCheckbox | Consent acceptance checkbox |
| QTableGrid | Grid/table input |
| QResponseMode | Response mode selector (upload / link / not available / will send later) |

### 8.4 Wizard Layout Components

| Component | Purpose |
|-----------|---------|
| WizardTopBar | Progress indicator, engagement branding, sign out |
| WizardSidebar | Section navigation (complete/partial/empty indicators) |
| WizardContextBanner | Contextual guidance for current section |
| WizardBottomBar | Previous / Save Draft / Next navigation |
| WizardRightPanel | Tips, help text, status |
| WizardSubmitSummary | Pre-submission readiness check |
| SubmitQuestionModal | In-wizard "ask a question" modal |

---

## 9. Email Notifications

Full specification in [notifications.md](notifications.md).

**Provider:** AWS SES (eu-west-2)
**Sender:** `portal@limitlessmodus.com` (Limitless Modus)

| Email Type | Trigger | Recipient | Status |
|------------|---------|-----------|--------|
| Pre-invitation | Consultant sends | Participant | Implemented |
| Invitation (instrument-specific) | Consultant sends | Participant | Implemented |
| Reminder | Consultant sends | Participant | Implemented |
| Completion / Thank You | Consultant sends | Participant | Implemented |

Email templates include instrument-specific variants for Director, Manager, and Engineer roles. Open/click tracking is implemented via tracking pixel and redirect endpoints.

See [notifications.md](notifications.md) for full template specification, DNS requirements, and gap analysis against Greg's email drafts.

---

## 10. Deployment

### 10.1 Infrastructure

| Component | Detail |
|-----------|--------|
| **Docker Hub** | `tumaigroup/limitless-portal` |
| **Host** | `bcl-limapp-10` |
| **Domain** | `portal.limitlessmodus.com` — Cloudflare DNS → HAProxy SSL → Go :8891 |
| **Supabase** | Active — EU region, 10 migrations applied, pilot data seeded |
| **AWS S3** | Active — eu-west-2, engagement-prefixed uploads |
| **AWS SES** | Active — eu-west-2, domain verified, DKIM/SPF/DMARC configured |
| **Azure SSO** | Implemented — needs end-to-end testing with real Microsoft accounts |
| **Google OAuth** | Implemented |

### 10.2 CI/CD Pipeline

Trigger: push to `main` or manual `workflow_dispatch`.

1. **Build job:** Docker multi-stage build (Go 1.24 + Node 22 → Ubuntu 24.04); push to Docker Hub with `latest` and timestamped tags
2. **Deploy job:** SSH to `bcl-limapp-10`; `docker compose pull && docker compose up -d`; health check

**Required GitHub secrets:** `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `DEVEXTREME_LICENSE_KEY`, `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_SSH_KEY`, `DEPLOY_PORT`

### 10.3 Deployment Files

| Path | Purpose |
|------|---------|
| `deploy/Dockerfile` | Multi-stage build (Go + Node → Ubuntu runtime) |
| `deploy/limitless-portal.service` | systemd unit file |
| `deploy/install.sh` | Pull from Docker Hub, extract binary + frontend + migrations |
| `.github/workflows/build.yml` | CI/CD pipeline |

**Known gap:** No `docker-compose.yml` in the repository. The CI/CD deploy step expects one to already exist at `/opt/limitless-portal/` on the target host.

---

## 11. Implementation Status

### 11.1 What Is Built

| Area | Detail |
|------|--------|
| Go backend scaffold | Config, router, 12 handler files, repository pattern, pgx/pgxpool |
| Authentication | Azure SSO + Google OAuth + dev-mode bypass + multi-engagement picker |
| Engagement management | Full CRUD, archive, slug, settings, logo, due dates |
| Participant management | CRUD, bulk import, instruments assignment, department/group |
| Company Audit wizard (01) | 14 steps, ~200 questions, 13 question components, consent, auto-save, submit |
| Manager Audit wizard (02) | Full wizard with department-conditional routing |
| Engineer Mini-Audit wizard (03) | Full wizard with job-type module selection |
| File uploads | S3 presigned URLs, direct browser upload, metadata in DB, in-wizard upload components |
| Communications | Pre-invite, invite, remind, completion emails via AWS SES; per-participant history |
| Email tracking | Open (pixel) + click (redirect) tracking with UI display |
| Consultant dashboard | Engagement grid, row navigation to audits |
| Participant dashboard | Assigned audits with progress bars, Start/Continue/View buttons |
| Submission reset | Admin danger-zone reset of submission to clean state |
| Database | 10 migrations, RLS on core tables, pilot seed data |
| CI/CD | GitHub Actions build + deploy to bcl-limapp-10 |

### 11.2 What Is Scaffolded / Partial

| Area | Gap |
|------|-----|
| Azure SSO end-to-end | Callback implemented but not yet tested with real Microsoft accounts |
| Evidence library | Upload flow works; no browse/search UI across all uploads for an engagement |
| Submission review (consultant) | Consultant can view submissions but no dedicated read-only review mode |
| Engagement creation UI | Settings page exists; initial creation flow could be smoother |

### 11.3 What Is Not Started

| Area | Notes |
|------|-------|
| Interview workflow | Pages are placeholder text only; no scheduling, transcript, or marker features |
| AI synthesis pipeline | Anthropic API key is in config; no synthesis service or handler code |
| Findings | Page is placeholder; no generation, review, or export |
| Report generation | No PDF/Word export of diagnostic findings |
| Scheduled email reminders | Manual send only; no cron/scheduled triggers |
| Auto-migration runner | Migrations applied manually |

---

## 12. Implementation Phases

### Phase 1 — Foundation (Current: substantially complete)

- [x] Go backend scaffold with repository pattern
- [x] Supabase schema (10 migrations)
- [x] AWS S3 file upload flow
- [x] Microsoft Azure SSO + Google OAuth
- [x] Engagement + participant management (CRUD, bulk import)
- [x] Vue 3 + DevExtreme frontend with SideNavOuterToolbar layout
- [x] Company Audit wizard (01) — full implementation
- [x] Manager Audit wizard (02) — full implementation
- [x] Engineer Mini-Audit wizard (03) — full implementation
- [x] File upload in wizards (presigned URL → S3 → confirm metadata)
- [x] Participant dashboard with progress tracking
- [x] Communications (email invite/remind/completion, tracking)
- [x] Deployment to portal.limitlessmodus.com (Docker + HAProxy + CI/CD)
- [ ] **Supabase upgrade to Pro tier** (critical for production)
- [ ] Azure SSO end-to-end test with real Microsoft accounts
- [ ] Add `.env.example` to repository
- [ ] Add `docker-compose.yml` to repository
- [ ] Add RLS policies to newer tables (003–010)
- [ ] Evidence library browse/search UI

### Phase 2 — Interview Workflow

- [ ] Interview scheduling (consultant creates slots, interviewee sees upcoming)
- [ ] Interview guide view (live script for Greg during calls)
- [ ] Transcript upload/capture (manual paste, file upload, ElevenLabs integration)
- [ ] Transcript marker tagging (manual + AI-assisted)
- [ ] 3-3-3 capture component (post-call summary)

### Phase 3 — AI Synthesis

- [ ] Synthesis pipeline (combine all submissions + transcripts → Claude API)
- [ ] Findings generation (constraints, breakpoints, cost leaks with TOC classification)
- [ ] Findings review + editing (consultant approves/refines)
- [ ] Cross-layer triangulation (director vs manager vs engineer contradictions)
- [ ] Coverage matrix dashboard (evidence themes across layers)

### Phase 4 — Reporting and Deliverables

- [ ] Diagnostic report generation (findings → structured document)
- [ ] Export to PDF/Word (using note-to-pdf / docx skills)
- [ ] Action plan generation (prioritised recommendations)
- [ ] Client-facing findings view (director sees approved findings)

---

## 13. Design Decisions Log

| # | Decision | Choice | Rationale | Date |
|---|----------|--------|-----------|------|
| D1 | Frontend framework | Vue 3 + DevExtreme + Vite + TypeScript | Standard Tumai webapp stack; wizard, data grid, form components ready-made | 2026-03-02 |
| D2 | Backend framework | Go (gorilla/mux) | Proven in node-agent + my-first-app; matches team expertise | 2026-03-02 |
| D3 | Primary auth | Microsoft Entra ID SSO | First client (Nano Fibre) uses Microsoft corporate accounts | 2026-03-02 |
| D4 | Secondary auth | Google OAuth | Flexibility for clients using Google Workspace | 2026-03-04 |
| D5 | Database | Supabase (managed PostgreSQL) | Cloud-hosted; managed backups, dashboard; SQLite rejected — not suited for multi-engagement platform | 2026-03-02 |
| D6 | DB abstraction | Repository pattern with `pgx` driver | Connection string swap between Supabase, local Postgres, BCL self-hosted | 2026-03-02 |
| D7 | File storage | AWS S3 | Pre-signed URLs avoid proxying large files; engagement-isolated prefixes | 2026-03-02 |
| D8 | Repo split | `limitless` (programme) + `limitless-portal-design` (design) + `limitless-portal` (code) | Clean separation of business strategy, product design, and implementation | 2026-03-02 |
| D9 | Domain | portal.limitlessmodus.com | Shared portal instance; engagement isolation at DB level, not subdomain level | 2026-03-05 |
| D10 | Email provider | AWS SES | Same AWS account as S3; shared IAM; London region for latency | 2026-03-06 |
| D11 | Email tracking | Custom pixel + redirect tracking | Simple, no third-party dependencies; tracking_token in notification_log | 2026-03-09 |
| D12 | Wizard auto-save | Section-level draft as JSONB | Entire section saved/loaded atomically; simpler than per-field auto-save | 2026-03-04 |
| D13 | Participant vs User tables | Separate tables | Participants are engagement-specific (role, department, instruments); Users are auth identities | 2026-03-04 |
| D14 | Multi-engagement support | EngagementPickerPage + SelectEngagement API | Users in multiple engagements choose which to work in | 2026-03-05 |

---

## 14. Open Questions and Risks

### Production Blockers

| # | Item | Status | Action Required |
|---|------|--------|-----------------|
| P1 | **Supabase Free tier** — no SLA, auto-pause, limited backups | **Critical** | Upgrade to Pro ($25/mo) before client onboarding |
| P2 | **Azure SSO not tested with real accounts** | High | End-to-end test with tumai.cc test accounts, then with Nano Fibre domain |
| P3 | **No `.env.example`** | Medium | Create with all required variables documented |
| P4 | **No `docker-compose.yml` in repo** | Medium | Add to deploy/ for reproducible deployment |
| P5 | **RLS gaps on newer tables** | Medium | Add policies for participants, section_drafts, support_questions, notification_log, email_events |

### Open Design Questions

| # | Question | Current Assumption | Priority |
|---|----------|-------------------|----------|
| Q1 | **Supabase region** | EU (Frankfurt) — verify it is correct | Before client onboarding |
| Q2 | **pgxpool max connections** | Default (no explicit limit) | Set explicit limit matching Supabase tier |
| Q3 | **Manual vs auto migration runner** | Manual psql | Automate before Phase 2 scaling |
| Q4 | **Offline support for engineers** | Online-only | Defer — reassess if connectivity issues arise |
| Q5 | **Transcript capture method** | Post-call manual upload | Phase 2 decision: manual vs ElevenLabs real-time |
| Q6 | **Notification scheduling** | Manual consultant-triggered only | Phase 2: add cron-based scheduled reminders |
| Q7 | **Multi-tenant data isolation** | Engagement-level isolation (shared DB, filtered queries) | Adequate for current scale; revisit if clients require physical isolation |

---

*Part of the [Limitless Portal Design](../README.md) — [Limitless Modus](../../README.md).*
