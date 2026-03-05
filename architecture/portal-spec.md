# Portal — Invincibility Blueprint™ Digital Platform

Planning, architecture, and design documentation for the web-based diagnostic portal that replaces the current Word docs + Microsoft Forms delivery.

**Code repository:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal)
**Live URL:** [nano.limitlessmodus.com](https://nano.limitlessmodus.com/)
**Toolkit specification:** [toolkit/](../toolkit/) (6 instruments — source of truth for what the portal implements)
**Legal framework:** [legal/](../legal/) (confidentiality, DPA, engagement terms baked into the portal)

---

## Architecture Overview

### Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | Vue 3 + DevExtreme | Proven Tumai webapp stack; wizard/stepper, data grids, form components map directly to audit instruments |
| **Backend** | Go (gorilla/mux) | Proven in node-agent and my-first-app; performant, typed, single-binary deployment |
| **Database** | Supabase (managed PostgreSQL) | Cloud-hosted, production-grade from day one; row-level security, connection pooling, dashboard for inspection |
| **File storage** | AWS S3 | Evidence uploads stored as raw artifacts; engagement-isolated prefixes, pre-signed URLs for direct browser upload |
| **Auth** | Microsoft Entra ID (Azure AD) SSO via OAuth2/OIDC | First client (Nano Fibre) uses Microsoft; domain-based login |
| **AI synthesis** | Anthropic API (Claude) | Transcript + audit data → diagnostic findings |
| **Deployment** | Docker + HAProxy on BCL | Same pattern as limitless-website |

### System Context

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
                                    │  nano.limitlessmodus.com │
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
    │ Supabase         │ │ AWS S3        │ │ Microsoft │ │ Anthropic        │ │ ElevenLabs   │
    │ (PostgreSQL)     │ │ (artifacts)   │ │ Entra ID  │ │ API (Claude)     │ │ (transcript) │
    │ Structured data  │ │ Raw uploads   │ │ (SSO)     │ │ (AI synthesis)   │ │              │
    └──────────────────┘ └───────────────┘ └───────────┘ └──────────────────┘ └──────────────┘
```

### Database Strategy

The Go backend uses a **database abstraction layer** (interface-based repository pattern) so the underlying database can be swapped without changing business logic. Supabase is the primary choice; alternatives are documented for future flexibility.

| Option | Type | Status | Use Case |
|--------|------|--------|----------|
| **Supabase** (managed PostgreSQL) | Cloud | **Active** — primary for production | Default for all engagements; managed backups, connection pooling, dashboard |
| Local PostgreSQL | Self-hosted | *Reserve* | Development, offline testing, air-gapped demos |
| BCL PostgreSQL (`bcl-postgres-60`) | Self-hosted | *Reserve* | On-premises option for clients requiring UK-only data residency |
| OWP / other self-hosted PostgreSQL | Self-hosted | *Reserve* | Client-hosted or partner-hosted environments |
| Other cloud DB (e.g., Neon, PlanetScale, AWS RDS) | Cloud | *Future* | Alternative managed providers if Supabase doesn't scale or pricing changes |

**Implementation approach:** The Go backend connects via standard `pgx` (PostgreSQL driver). The connection string is the only thing that changes between environments — same schema, same migrations, same queries. A `DB_PROVIDER` environment variable selects the active configuration.

### File Storage Strategy

Raw artifacts (evidence uploads, transcripts, generated reports) are stored separately from the database.

| Option | Type | Status | Use Case |
|--------|------|--------|----------|
| **AWS S3** | Cloud | **Active** — primary for production | Default for all engagements; pre-signed URLs for direct browser upload |
| Local filesystem | Self-hosted | *Reserve* | Development, testing |
| BCL / OWP NFS/block storage | Self-hosted | *Reserve* | On-premises option for data residency requirements |

**Bucket structure:** `s3://limitless-portal/{engagement-id}/{instrument}/{filename}` — each engagement is fully isolated at the prefix level.

---

## User Roles

| Role | Who | Portal Access |
|------|-----|---------------|
| **Consultant** | Greg Kurnikov | Full access — manage engagements, view all submissions, run interviews, trigger synthesis, view reports |
| **Admin** | Fedor (system) | System configuration, user management, engagement setup |
| **Director** | Client director/MD | Submit Company Audit (01), view engagement progress, view findings/reports |
| **Manager** | Client functional managers | Submit Manager Audit (02), view own submission status |
| **Engineer** | Client field engineers | Submit Engineer Mini-Audit (03), minimal UI — survey only |

### Auth Flow (SSO)

> **Full auth documentation:** [auth/](../auth/) — SSO strategy, login page design, test accounts, role model, decisions log.

```
User visits nano.limitlessmodus.com
  → Login page shows SSO buttons (Microsoft / Google)
  → Redirected to identity provider
  → Authenticates with their corporate account
  → Redirected back with OAuth2 authorization code
  → Go backend exchanges code for tokens, parses email
  → Looks up email in participants → users table
  → JWT issued for session management
  → Role determined by database record
```

---

## Data Model (Conceptual)

### Core Entities

```
Engagement (1 per client)
  ├── id, client_name, client_legal_name, status, created_at
  ├── supplier_name, supplier_legal_name
  ├── config (placeholders: [name/email], etc.)
  │
  ├── Users (N per engagement, each with a role)
  │   └── id, engagement_id, email, name, role, azure_oid, invited_at, last_login
  │
  ├── Submissions (N per engagement — one per user per instrument)
  │   ├── id, engagement_id, user_id, instrument_type (01–06)
  │   ├── status (draft / submitted / reviewed)
  │   ├── started_at, submitted_at, reviewed_at
  │   │
  │   └── Responses (N per submission — one per field/question)
  │       ├── id, submission_id, section, field_key
  │       ├── response_type (text / choice / multi_choice / rating / file)
  │       ├── value_text, value_json
  │       └── updated_at (for auto-save)
  │
  ├── Uploads (N per engagement)
  │   ├── id, engagement_id, submission_id (nullable), user_id
  │   ├── filename, mime_type, size_bytes, storage_path
  │   ├── section, field_key (which question it answers)
  │   └── uploaded_at
  │
  ├── Interviews (N per engagement)
  │   ├── id, engagement_id, interviewee_id, instrument_type (04–06)
  │   ├── status (scheduled / completed / transcribed / synthesised)
  │   ├── scheduled_at, completed_at
  │   ├── transcript_text, transcript_source (manual / elevenlabs / upload)
  │   │
  │   └── Markers (N per interview — AI synthesis tags)
  │       ├── id, interview_id, marker_type (constraint / break_point / cost_leak / ...)
  │       ├── text, timestamp_offset
  │       └── tagged_by (auto / manual)
  │
  └── Findings (N per engagement — AI-synthesised outputs)
      ├── id, engagement_id, finding_type (constraint / breakpoint / cost_leak / ...)
      ├── title, description, evidence_refs
      ├── toc_dimension (throughput / inventory / operating_expense)
      ├── severity (critical / high / medium / low)
      ├── sources (which submissions/interviews contributed)
      └── created_at
```

### Instrument Types

| Code | Instrument | Submission Mode |
|------|-----------|-----------------|
| `01` | Company Audit | Multi-section wizard, auto-save, file uploads |
| `02` | Manager Audit | Multi-section wizard with department-conditional routing |
| `03` | Engineer Mini-Audit | Short survey wizard with job-type module selection |
| `04` | Director Interview | Scheduled by consultant; transcript captured post-call |
| `05` | Manager Interview | Scheduled by consultant; transcript captured post-call |
| `06` | Engineer Interview | Scheduled by consultant; transcript captured post-call |

---

## Key UX Patterns

### Audit Wizard

Each audit instrument (01–03) is delivered as a **multi-step wizard** using DevExtreme Stepper or Wizard component:

```
┌──────────────────────────────────────────────────────────┐
│  Company Audit — Section B: Organisation Structure       │
│                                                          │
│  ● Consent  ● Context  ● A  ◉ B  ○ C  ○ D  ○ E ...    │
│  ─────────────────────────────────────────────────────── │
│                                                          │
│  B1. Org Chart & Team List                               │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Upload the latest organisation chart               │  │
│  │ ┌──────────────────┐                               │  │
│  │ │  📎 Drop file    │  org-chart.pdf (2.1 MB) ✓    │  │
│  │ └──────────────────┘                               │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  B2. Regular Duties & Cadence (per department)           │
│  ┌────────────────────────────────────────────────────┐  │
│  │ ☑ Installs          uploaded: duties-installs.xlsx │  │
│  │ ☑ Service Calls     uploaded: duties-service.xlsx  │  │
│  │ ☐ Pre-Enablement    Not available yet              │  │
│  │ ☐ Enablement Works  Exists but needs exporting     │  │
│  │   ETA: [___________]                               │  │
│  │ ...                                                │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│              [← Previous]  [Save Draft]  [Next →]        │
└──────────────────────────────────────────────────────────┘
```

**Key features:**
- Auto-save on every field change (debounced)
- Section progress indicators (complete / partial / empty)
- Upload guidance shown contextually (from Section 0)
- "For every request, choose one" pattern (upload / link / not available / will send later)
- Department-conditional routing in Manager Audit (Section 5)
- Job-type module selection in Engineer Mini-Audit (Section 5)

### Reality Test Rating Grid

A reusable component for Section E2 (Company Audit) and Section 10 (Manager Audit):

```
┌──────────────────────────────────────────────────────────┐
│  Rating: Planning & scheduling                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │ ○ Exists and is used                               │  │
│  │ ○ Exists but not used consistently                 │  │
│  │ ○ Claimed to exist but cannot be found             │  │
│  │ ○ Exists but under-resourced                       │  │
│  │ ○ Implemented but no effect                        │  │
│  │ ○ Tool used for wrong purpose                      │  │
│  │ ○ Multiple approaches conflict                     │  │
│  │ ○ Works only because specific people carry it      │  │
│  └────────────────────────────────────────────────────┘  │
│  Evidence: [________________________________]            │
│  Upload:   [📎 Drop file]                                │
└──────────────────────────────────────────────────────────┘
```

### Engagement Dashboard (Consultant View)

```
┌──────────────────────────────────────────────────────────┐
│  Nano Fibre UK — Diagnostic Progress                     │
│                                                          │
│  Phase 2: Self-Submission          ██████████░░ 73%      │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 01 Company Audit    [Director]   ████████████ 100% │  │
│  │ 02 Manager Audit    [Ops Mgr]    ██████████░░  80% │  │
│  │ 02 Manager Audit    [QA Mgr]     ████████░░░░  60% │  │
│  │ 02 Manager Audit    [Stores]     ██░░░░░░░░░░  15% │  │
│  │ 03 Engineer Survey  [4 of 8]     ████░░░░░░░░  50% │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Phase 3: Interviews               ██░░░░░░░░░░ 17%     │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 04 Director Interview  Scheduled: 5 Mar 10:00     │  │
│  │ 05 Manager Interviews  0 of 4 completed           │  │
│  │ 06 Engineer Interviews 0 of 3 completed           │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Evidence Library: 23 files (142 MB)  [View All →]       │
└──────────────────────────────────────────────────────────┘
```

---

## API Design (Draft)

### Authentication

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/auth/login` | — | Redirect to Microsoft Entra ID |
| GET | `/auth/callback` | — | OAuth2 callback, issue JWT |
| POST | `/auth/refresh` | JWT | Refresh access token |
| POST | `/auth/logout` | JWT | Revoke session |
| GET | `/auth/me` | JWT | Current user + role + engagement |

### Engagements

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements` | Admin/Consultant | List all engagements |
| POST | `/api/engagements` | Admin | Create engagement |
| GET | `/api/engagements/:id` | Member | Get engagement details + progress |
| PUT | `/api/engagements/:id` | Admin/Consultant | Update engagement config |
| GET | `/api/engagements/:id/users` | Admin/Consultant | List engagement users |
| POST | `/api/engagements/:id/users` | Admin/Consultant | Invite user to engagement |

### Submissions (Audits)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements/:id/submissions` | Consultant | List all submissions for engagement |
| GET | `/api/submissions/:id` | Owner/Consultant | Get submission with all responses |
| POST | `/api/submissions` | Member | Create submission (start an audit) |
| PUT | `/api/submissions/:id/responses` | Owner | Save/update field responses (auto-save) |
| POST | `/api/submissions/:id/submit` | Owner | Mark submission as complete |
| GET | `/api/submissions/:id/progress` | Owner/Consultant | Section-level completion status |

### Uploads (S3-backed)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/uploads/presign` | Member | Generate S3 pre-signed upload URL (browser uploads directly to S3) |
| POST | `/api/uploads/confirm` | Member | Confirm upload completed, save metadata to DB |
| GET | `/api/uploads/:id/url` | Member (same engagement) | Generate S3 pre-signed download URL |
| GET | `/api/engagements/:id/uploads` | Consultant | List all upload metadata for engagement |
| DELETE | `/api/uploads/:id` | Owner/Consultant | Remove from S3 + delete metadata |

### Interviews

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/engagements/:id/interviews` | Consultant | List interviews |
| POST | `/api/interviews` | Consultant | Schedule interview |
| PUT | `/api/interviews/:id` | Consultant | Update status/transcript |
| POST | `/api/interviews/:id/transcript` | Consultant | Upload/set transcript text |
| GET | `/api/interviews/:id/markers` | Consultant | Get synthesis markers |
| POST | `/api/interviews/:id/markers` | Consultant/System | Add markers (manual or AI) |

### Synthesis & Findings

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/engagements/:id/synthesis` | Consultant | Trigger AI synthesis run |
| GET | `/api/engagements/:id/findings` | Consultant | List generated findings |
| PUT | `/api/findings/:id` | Consultant | Edit/approve finding |

---

## Implementation Phases

### Phase 1 — Foundation *(MVP for Nano Fibre engagement)*

- [ ] Go backend scaffold (config, router, middleware, repository pattern with pgx)
- [ ] Supabase project setup (schema, migrations, connection pooling, RLS policies)
- [ ] AWS S3 bucket setup (engagement-isolated prefixes, pre-signed URL generation)
- [ ] Microsoft Azure SSO integration (OAuth2/OIDC with Entra ID)
- [ ] Engagement + user management (CRUD, role assignment, invitation)
- [ ] Vue 3 + DevExtreme frontend scaffold (layouts, auth store, router)
- [ ] Company Audit wizard (01) — all 11 sections with auto-save
- [ ] File upload flow (pre-signed URL → S3 direct upload → confirm metadata)
- [ ] Engagement dashboard (consultant view — progress tracking)
- [ ] Deployment to nano.limitlessmodus.com (Docker + HAProxy)

### Phase 2 — Full Audit Suite

- [ ] Manager Audit wizard (02) — with department-conditional routing
- [ ] Engineer Mini-Audit wizard (03) — with job-type module selection
- [ ] Evidence library (browse/search uploaded files per engagement)
- [ ] Submission review view (consultant reads completed audits)
- [ ] Email notifications (invitation, reminder, submission complete)

### Phase 3 — Interview Workflow

- [ ] Interview scheduling (consultant schedules, interviewee sees upcoming)
- [ ] Interview guide view (live script for Greg during calls)
- [ ] Transcript upload/capture (manual paste, file upload, ElevenLabs integration)
- [ ] Transcript marker tagging (manual + AI-assisted)
- [ ] 3-3-3 capture component (post-call summary)

### Phase 4 — AI Synthesis

- [ ] Synthesis pipeline (combine all submissions + transcripts → Claude API)
- [ ] Findings generation (constraints, breakpoints, cost leaks with TOC classification)
- [ ] Findings review + editing (consultant approves/refines)
- [ ] Cross-layer triangulation (director vs manager vs engineer contradictions)
- [ ] Coverage matrix dashboard (which themes have evidence from which layers)

### Phase 5 — Reporting & Deliverables

- [ ] Diagnostic report generation (findings → structured document)
- [ ] Export to PDF/Word (using note-to-pdf / docx skills)
- [ ] Action plan generation (prioritised recommendations)
- [ ] Client-facing findings view (director sees approved findings)

---

## Design Decisions Log

| # | Decision | Choice | Rationale | Date |
|---|----------|--------|-----------|------|
| D1 | Frontend framework | Vue 3 + DevExtreme | Standard Tumai webapp stack; wizard, data grid, form components ready-made | 2026-03-02 |
| D2 | Backend framework | Go (gorilla/mux) | Proven in node-agent + my-first-app; matches team expertise | 2026-03-02 |
| D3 | Authentication | Microsoft Entra ID SSO | First client (Nano Fibre) uses Microsoft corporate accounts | 2026-03-02 |
| D4 | Database | Supabase (managed PostgreSQL) | Cloud-hosted production-grade DB from day one; ~~SQLite~~ rejected — not suited for cloud-first multi-engagement platform | 2026-03-02 |
| D5 | Repo split | `limitless` (planning) + `limitless-portal` (code) | Same pattern as kseniia-academy + kseniia-academy-webapp | 2026-03-02 |
| D6 | Subdomain | nano.limitlessmodus.com | Client-branded; "nano" for the Nano Fibre engagement | 2026-03-02 |
| D7 | Instrument spec ownership | Toolkit markdown in `limitless` repo | Instruments are methodology, not code — spec lives in programme repo | 2026-03-02 |
| D8 | File storage | AWS S3 | Raw artifacts stay out of the database; pre-signed URLs avoid proxying large files through the Go backend | 2026-03-02 |
| D9 | DB abstraction | Repository pattern (interface-based) with `pgx` driver | Connection string is the only change between Supabase, local Postgres, BCL self-hosted, or other providers | 2026-03-02 |

---

## Open Questions

- [ ] **Multi-tenancy model:** One portal instance per client (subdomain) or shared instance with engagement isolation? Current assumption: shared instance, engagement-level isolation in Supabase (RLS policies per engagement).
- [ ] **Transcript capture:** Manual upload only, or integrate real-time transcription (ElevenLabs / Azure Speech)? Current assumption: post-call upload for Phase 1.
- [ ] **S3 bucket per client vs shared:** Single shared bucket with prefix isolation, or one bucket per engagement? Current assumption: shared bucket, prefix-isolated (`{engagement-id}/`).
- [ ] **Supabase project region:** EU (Frankfurt) for UK data residency, or US default? Decision needed before project creation.
- [ ] **Offline / partial submission:** Do we need offline capability for engineers with poor connectivity? Current assumption: no, online-only for Phase 1.
- [ ] **Notification channel:** Email only, or also Microsoft Teams integration? Current assumption: email for Phase 1.
- [ ] **DB migration tooling:** Use Go-native migrations (golang-migrate, goose) or Supabase CLI migrations? Current assumption: Go-native for portability across DB providers.

---

*Part of the [Invincibility Blueprint](../README.md) — [Limitless Modus](../../README.md).*
