# Portal — Where We Are & Action Plan

*Last updated: 2026-03-04 (evening — post Company Audit Wizard deployment)*

**Code repo:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal)
**Live URL:** [nano.limitlessmodus.com](https://nano.limitlessmodus.com/)
**Architecture:** [README.md](README.md) (stack, data model, API design, phases)
**Toolkit spec:** [toolkit/](../toolkit/) (6 instruments — source of truth)

---

## Current State

The portal has a **working dev environment** with auth bypass, a dashboard, and a fully implemented Company Audit Wizard. The layout shell was fixed on 2026-03-04 (SideNavOuterToolbar pattern ported from Node Agent). The Company Audit Wizard (Instrument 01) with 14 steps, ~200 questions, 13 reusable question components, participant API, and dual-role dev login was deployed the same day. CI/CD auto-deploys to `bcl-limapp-10` on push to main.

See [2026-03-04 progress update](../../working/updates/2026-03-04-2400-company-audit-wizard-deployed.md) for the full session log.

### What Works

| Area | Status | Detail |
|------|--------|--------|
| **Go backend scaffold** | Done | Config, router, middleware (JWT + API key + CORS + logging), repository pattern with `pgx` |
| **Dev-mode auth** | Done | When `JWT_SECRET` starts with `dev-`, login bypasses Azure SSO — enter any email, get a JWT |
| **Engagement list/get** | Done | `GET /api/engagements`, `GET /api/engagements/:id` — reads from Postgres |
| **Submission CRUD** | Done | List, Get, Create, GetResponses, SaveResponse — all wired to Postgres |
| **Vue frontend scaffold** | Done | Vue 3 + DevExtreme + TypeScript + Vite, SideNavOuterToolbar layout, Pinia auth store |
| **Dashboard page** | Done | DevExtreme DataGrid showing engagements, welcome card, row click navigates to Audits |
| **Audit wizard (Company Audit)** | Done | **Full 14-step wizard** (Consent → A–K → Review → Thank You), ~200 questions, 13 reusable question types, auto-save, progress tracking |
| **Participant dashboard** | Done | Landing page after SSO — shows assigned audits with status, progress bar, Start/Continue buttons |
| **Participant API** | Done | 6 endpoints: dashboard, get submission, save section draft, consent, submit, update progress |
| **Wizard layout** | Done | Separate from admin layout: WizardTopBar, WizardContextBanner, WizardSidebar, WizardBottomBar |
| **Dual dev login** | Done | Two buttons on login page: "Dev Sign In (Consultant)" and "Dev Sign In (Participant)" with role-based routing |
| **Audits page** | Done | Shows instrument cards (01–03), submissions grid, start/resume audit flow |
| **Login page** | Done | Dev login (email + button) + Microsoft sign-in button (redirects to `/auth/login`) |
| **Database schema** | Done | `001_initial_schema.up.sql` + `002_enable_rls.up.sql` + **`003_wizard_data_model.up.sql`** (participants, section_drafts, altered submissions, seeded pilot data) |
| **RLS policies** | Done | `migrations/002_enable_rls.up.sql` — row-level security for Supabase |
| **Deployment pipeline** | Done | Dockerfile (multi-stage), GitHub Actions build+push+deploy, systemd service, install.sh |
| **Layout (fixed 2026-03-04)** | Done | SideNavOuterToolbar with responsive DxDrawer, HeaderToolbar, SideNavMenu (DxTreeView) |

### What's Scaffolded / Partial

| Area | Status | Gap |
|------|--------|-----|
| **Azure SSO** | Implemented | Callback handler exchanges authorization code, extracts email from id_token, looks up participant/user, issues JWT. **Not yet tested with real Microsoft accounts** — 6 tumai.cc test accounts created, see [auth/microsoft-sso.md](../auth/microsoft-sso.md) |
| **Engagement creation** | Handler exists | Returns a placeholder message — no actual DB insert |
| **S3 client** | Service file exists (`services/s3.go`) | No upload handler endpoints in the router, no presign flow |
| **Interviews page** | Route exists | Placeholder text only — no UI or API integration |
| **Findings page** | Route exists | Placeholder text only — no UI or API integration |

### What's Not Started

| Area | Notes |
|------|-------|
| **Manager Audit wizard (02)** | Frontend only has Company Audit form sections |
| **Engineer Mini-Audit wizard (03)** | Not started |
| **File upload flow** | S3 presign endpoints, browser direct upload, confirm metadata |
| **Interview workflow** | Scheduling, transcript capture, marker tagging |
| **AI synthesis pipeline** | Anthropic API integration, findings generation |
| **Email notifications** | Backend service + API endpoints done (AWS SES). See [notifications.md](notifications.md). UI wiring pending (Phase 1C/2) |
| **Report generation** | PDF/Word export of diagnostic findings |
| **Evidence library** | Browse/search uploaded files per engagement |

### Known Issues

| # | Issue | Impact | Priority |
|---|-------|--------|----------|
| 1 | **Instrument type mismatch** | Frontend sends `01-company-audit` but DB CHECK constraint expects `company_audit` — creating a submission will fail | **Critical** |
| 2 | **No migration runner** | Migrations must be applied manually via `psql` or Supabase dashboard — no `make migrate` or auto-run on startup | Medium |
| 3 | **No `docker-compose.yml` in repo** | Deploy step SSHs to server and expects `docker-compose.yml` to already exist at `/opt/limitless-portal/` | Low (works, but fragile) |
| 4 | **No `.env.example`** | New developers have no reference for required environment variables | Low |
| 5 | **Audit wizard sections vs toolkit spec** | Current 9-tab form is a generic scaffold — not yet mapped 1:1 to the 11 sections (A–K) in the Company Audit toolkit spec | Medium |

---

## Action Plan

### Phase 1A — Make What Exists Actually Work (Priority: NOW)

These items fix the current scaffolded code so the portal can be used for a real demo or early engagement.

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 1.1 | **Fix instrument type mapping** — align frontend instrument codes with DB CHECK constraints (either change the schema or map in the frontend/backend) | Small | None |
| 1.2 | **Implement engagement creation** — wire `POST /api/engagements` handler to actually insert into Postgres | Small | None |
| 1.3 | **Implement Azure SSO callback** — exchange authorization code for tokens, parse ID token, lookup/create user in DB, issue JWT, redirect to SPA | Medium | Azure app registration configured |
| 1.4 | **Add `.env.example`** to the repo with all required variables documented | Small | None |
| 1.5 | **Add `docker-compose.yml`** to the repo for the production deployment pattern | Small | None |
| 1.6 | **Add Supabase project + apply migrations** — create the Supabase project (EU region), apply both migration files, verify schema | Small | Supabase account |
| 1.7 | **Test end-to-end flow** — dev login → create engagement → start Company Audit → fill sections → save responses → verify data in Supabase | Medium | 1.1, 1.2, 1.6 |

### Phase 1B — Company Audit Completeness

Map the audit wizard to the actual toolkit spec so it matches what Greg delivers.

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 1.8 | **Map Company Audit sections** — align the 9-tab frontend form with the 11 sections (A–K) from [01-company-audit.md](../toolkit/01-company-audit.md), including all field types and upload prompts | Large | 1.7 working |
| 1.9 | **Add file upload flow** — S3 presign endpoints, browser direct upload, confirm metadata in DB, show uploaded files in form | Medium | S3 bucket configured |
| 1.10 | **Add submission progress tracking** — section-level completion indicators (complete / partial / empty) in the wizard stepper | Small | 1.8 |
| 1.11 | **Add submission review view** — read-only consultant view of a completed audit with all responses and uploads | Medium | 1.8, 1.9 |

### Phase 1C — Multi-User & Engagement Management

Enable the consultant (Greg) to manage engagements and invite client users.

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 1.12 | **Engagement management UI** — create/edit engagements, configure client details, manage users | Medium | 1.2 |
| 1.13 | **User invitation flow** — consultant invites client users by email, assigns roles (Director / Manager / Engineer) | Medium | 1.3 (Azure SSO) |
| 1.14 | **Role-based access** — restrict views and actions based on user role (consultant sees all, director sees own engagement, manager sees own submissions) | Medium | 1.13 |
| 1.15 | **Engagement dashboard** — progress tracking per instrument per user (the mockup in README.md) | Medium | 1.8, 1.14 |

### Phase 2 — Full Audit Suite

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 2.1 | **Manager Audit wizard (02)** — 13 sections with department-conditional routing (Section 5) | Large | 1.8 pattern established |
| 2.2 | **Engineer Mini-Audit wizard (03)** — 10 sections with job-type module selection (Section 5) | Medium | 1.8 pattern established |
| 2.3 | **Reality Test rating grid** — reusable component for Section E2 (Company) and Section 10 (Manager) | Small | 2.1 |
| 2.4 | **Email notifications** — backend done (AWS SES); wire invitation UI to API, auto-send on submission | Small | 1.13 |

### Phase 3 — Interview Workflow

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 3.1 | **Interview scheduling** — consultant creates interview slots, interviewee sees upcoming | Medium | 1.14 |
| 3.2 | **Interview guide view** — live script for Greg during calls (from toolkit specs 04–06) | Medium | None |
| 3.3 | **Transcript capture** — manual paste, file upload, ElevenLabs integration | Medium | 1.9 (upload flow) |
| 3.4 | **Transcript marker tagging** — manual + AI-assisted using synthesis markers from toolkit | Medium | 3.3 |
| 3.5 | **3-3-3 capture component** — post-call summary (3 constraints, 3 breakpoints, 3 cost leaks) | Small | 3.4 |

### Phase 4 — AI Synthesis

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 4.1 | **Synthesis pipeline** — combine all submissions + transcripts, send to Claude API | Large | Phase 2 + 3 |
| 4.2 | **Findings generation** — constraints, breakpoints, cost leaks with TOC classification | Large | 4.1 |
| 4.3 | **Findings review UI** — consultant approves/refines AI-generated findings | Medium | 4.2 |
| 4.4 | **Cross-layer triangulation** — highlight director vs manager vs engineer contradictions | Medium | 4.2 |
| 4.5 | **Coverage matrix dashboard** — which themes have evidence from which layers | Medium | 4.2 |

### Phase 5 — Reporting

| # | Task | Effort | Dependencies |
|---|------|--------|--------------|
| 5.1 | **Diagnostic report generation** — findings → structured markdown | Medium | 4.3 |
| 5.2 | **Export to PDF/Word** — using note-to-pdf / docx skills | Small | 5.1 |
| 5.3 | **Action plan generation** — prioritised recommendations from findings | Medium | 4.3 |
| 5.4 | **Client-facing findings view** — director sees approved findings in the portal | Medium | 4.3, 1.14 |

---

## Deployment Status

| Component | Status | Detail |
|-----------|--------|--------|
| **Docker Hub** | `tumaigroup/limitless-portal` | Auto-built on push to main via GitHub Actions |
| **Host** | `bcl-limapp-10` | Auto-deployed after Docker push (SSH + docker compose) |
| **Domain** | `nano.limitlessmodus.com` | Cloudflare DNS → HAProxy SSL → Go backend :8891 |
| **Supabase** | Active | Project provisioned, migrations 001–003 applied, pilot data seeded |
| **AWS S3** | Not yet provisioned | Need to create bucket, configure CORS, set IAM credentials |
| **Azure SSO** | Implemented | Callback handler built — needs end-to-end testing with real Microsoft accounts. Login page redesigned: dev bypass removed, SSO-only. See [auth/](../auth/) |

---

## Decisions Pending

| # | Question | Options | Leaning |
|---|----------|---------|---------|
| Q1 | Supabase region | EU (Frankfurt) for UK data residency vs US default | EU — Nano Fibre is UK-based |
| Q2 | S3 bucket strategy | One shared bucket with prefix isolation vs one bucket per client | Shared bucket with `{engagement-id}/` prefix |
| Q3 | Migration runner | Manual `psql` vs Go auto-migrate on startup vs Supabase CLI | Go auto-migrate for portability |
| Q4 | Instrument type alignment | Change DB CHECK constraint to match frontend codes, or map in handler | Map in handler (keep DB clean, frontend human-readable) |
| Q5 | Offline support for engineers | Online-only vs PWA with offline queue | Online-only for Phase 1 |

---

## Quick Reference

```
Code:     C:\Users\fedor\repos\tumai-programmes\limitless-portal
Planning: C:\Users\fedor\repos\tumai-programmes\limitless\methodology\invincibility-blueprint\portal\
Toolkit:  C:\Users\fedor\repos\tumai-programmes\limitless\methodology\invincibility-blueprint\toolkit\
```

| Command | What it does |
|---------|-------------|
| `make dev` | Run Go backend locally (hot reload: restart manually) |
| `cd frontend && npm run dev` | Run Vue frontend dev server (proxies API to :8891) |
| `make all` | Build Go binary + Vue frontend |
| `git push` | Triggers CI/CD: build Docker → push to Docker Hub → auto-deploy to bcl-limapp-10 |

---

*Part of the [Portal Architecture](README.md) — [Invincibility Blueprint](../README.md) — [Limitless Modus](../../README.md).*
