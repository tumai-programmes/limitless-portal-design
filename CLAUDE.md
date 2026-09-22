*Last updated: 2026-09-22 12:27 (UK)*

# CLAUDE.md — Limitless Portal Design

Primary instructions and context for Claude when working in this repository.

## Project Overview

**Limitless Portal Design** is the product planning, UI design, architecture specification, and roadmap repository for the Limitless Modus Portal — an AI-powered operational transformation SaaS platform. This repo separates product design concerns from both the business programme (`limitless`) and the code implementation (`limitless-portal`).

**Repository:** https://github.com/tumai-programmes/limitless-portal-design
**Organisation:** tumai-programmes (Programme Tier)
**Code repo:** `../limitless-portal/` — [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal) (Go + Vue implementation)
**Programme repo:** `../limitless/` — [tumai-programmes/limitless](https://github.com/tumai-programmes/limitless) (business strategy, methodology, engagements)
**SQL operations:** [tumai-hq/sql-hub/limitless-portal](https://github.com/tumai-hq/sql-hub) (monitoring, diagnostics, reports)
**Live URL:** https://portal.limitlessmodus.com/
**Dev URL:** https://dev-portal.limitlessmodus.com/

## Limitless Portal Triad

The Limitless Modus portal is developed across three dedicated repositories, each with a distinct role:

| Repo | Role | What belongs here |
|------|------|-------------------|
| [`limitless`](https://github.com/tumai-programmes/limitless) | **Programme** — business strategy, methodology, engagements | Greg's operational methodology, partnership terms, market research, brand identity, client engagement materials, commercial service definition |
| **`limitless-portal-design`** (this repo) | **Design** — product design, UI/UX, architecture, style system | Figma references, wizard UI layouts, component specs, design tokens (fonts, colours, spacing), architecture decisions, roadmap |
| [`limitless-portal`](https://github.com/tumai-programmes/limitless-portal) | **Code** — implementation, deployment-ready artefacts | Pure code written primarily by AI (Claude Code); Go backend, Vue frontend, CI/CD, Docker, migrations — minimal manual human touch |

**Boundary rule:** If it is about *what the business does* → `limitless`. If it is about *how the product looks and is planned* → `limitless-portal-design`. If it is *deployable code* → `limitless-portal`.

## Shared Resources

| Resource | Location |
|----------|----------|
| Skills (121) | `../../tumai-hq/skills/` - [tumai-hq/skills](https://github.com/tumai-hq/skills) |
| API Credentials | `~/repos/tumai-hq/.mindatlas/credentials/.env` (local only) |
| Shared Config | `~/repos/tumai-hq/.mindatlas/config/` |
| Domain Config | `./config/domain.yaml` |
| Programme knowledge | `../limitless/` — methodology, architecture, engagements |
| Methodology instruments | `../limitless/methodology/invincibility-blueprint/toolkit/` |
| Legal templates | `../limitless/methodology/invincibility-blueprint/legal/` |
| Portal code | `../limitless-portal/` — Go backend, Vue frontend, migrations, deployment |
| Public website | `../limitless-website/` — Nuxt SSG at limitlessmodus.com |
| Webapp stack reference | `../../tumai-hq/it-hub/technology-stack-webapps/` |
| Frontend layouts | `../../tumai-hq/it-hub/technology-stack-webapps/frontend-layouts/` |
| DevExtreme components | `../../tumai-hq/it-hub/technology-stack-webapps/devextreme-components/` |

## File Locations

| Path | Purpose |
|------|---------|
| `architecture/` | Portal architecture spec, system context, data model, API design, status tracker |
| `architecture/portal-spec.md` | Full architecture specification |
| `architecture/status.md` | Current development status and action plan |
| `architecture/notifications.md` | Email notification design spec (AWS SES, templates, API) |
| `design-tokens/` | **Design system source of truth** — colour palette, typography, spacing, radii, shadows |
| `design-tokens/colours.md` | Brand palette, semantic colours, neutral scale, surfaces & borders |
| `design-tokens/typography.md` | Font family, type scale, font weights |
| `design-tokens/spacing.md` | Spacing scale, layout constants, border radii, box shadows |
| `wizard/` | Wizard UI specifications — per-instrument layout designs |
| `wizard/images/` | Wizard screenshots, architecture diagrams, HMRC reference (66 images) |
| `deliverables/` | Formatted outputs (PDF, DOCX) of wizard specs and plans |
| `working/` | Session notes and progress updates |
| `config/` | Domain configuration |

## Technology Stack (Portal)

| Layer | Technology | Notes |
|-------|-----------|-------|
| **Frontend** | Vue 3 (Composition API) + TypeScript | SPA served as static files by Go backend |
| **UI Components** | DevExtreme (Vue edition) | Wizard/stepper, data grids, form editors, file upload |
| **Backend** | Go (gorilla/mux) | REST API for auth, submissions, uploads, synthesis |
| **Database** | Supabase (managed PostgreSQL) via `pgx` driver | Repository pattern allows swapping providers |
| **File Storage** | AWS S3 | Raw artifact uploads; pre-signed URLs for direct browser upload |
| **Auth** | Microsoft Entra ID (Azure AD) SSO | OAuth2/OIDC; JWT for session management |
| **AI Synthesis** | Anthropic API (Claude) | Transcript + audit data → diagnostic findings |
| **Design** | Figma | Source of truth for visual design |
| **Deployment** | Docker + HAProxy + GitHub Actions | BCL self-hosted infrastructure |

## Wizard UI Specifications

Each audit instrument (01–03) has been designed as a multi-step wizard with DevExtreme component mapping:

| Spec | Instrument | Status |
|------|-----------|--------|
| `wizard/specification.md` | Overall wizard specification | Complete |
| `wizard/01-company-audit-layouts.md` | Company Audit (01) — 11 sections, ~200 questions | Complete, implemented |
| `wizard/02-manager-audit-layouts.md` | Manager Audit (02) — 13 sections with dept routing | Complete, not yet implemented |
| `wizard/03-engineer-mini-audit-layouts.md` | Engineer Mini-Audit (03) — 10 sections with job-type modules | Complete, not yet implemented |

## Boundary Rules

- **"Is it about what the business does — methodology, partnerships, research?"** → `limitless/`
- **"Is it about how the portal looks, works, or is architected?"** → here (`limitless-portal-design/`)
- **"Is it deployable code, migrations, or deploy config?"** → `limitless-portal/`
- **"Is it a SQL query for monitoring or analysis?"** → `sql-hub/limitless-portal/`

## Available Skills

Skills are loaded from `../../tumai-hq/skills/` ([tumai-hq/skills](https://github.com/tumai-hq/skills)):

| Category | Skills |
|----------|--------|
| **Documents** | pdf, pdf-form-filler, docx, pptx, pptx-to-pdf, xlsx, book-to-markdown, ch-accounts-to-markdown, report-to-markdown, note-to-pdf, note-to-word, export-images |
| **Research** | article-reflection, source-digest, linkedin, twitter |
| **Finance** | xero, xero-ap-invoice, rent-invoice, rent-invoice-approve, hmrc, tinytax, tumai-management-payroll-close, invoice-pdf, vat-invoice-request, fportal-prompt, n8n-ap-registry, edgematics-invoice, edgematics-agreement, cleaning-bill-ref |
| **Airtable** | airtable |
| **Atlassian** | jira, atlassian-discovery, atlassian-goals, atlassian-projects |
| **Confluence** | confluence-sync, confluence-scan, confluence-summarise, confluence-publisher, confluence-manager, confluence-pull, confluence-email-scan |
| **Google** | gmail, gcalendar, gdocs, gsheets, gdrive-sync, gworkspace-admin, google-ads, google-analytics, youtube, youtube-notes |
| **SEO** | bing-webmaster |
| **Infrastructure** | cloudflare, statuscake, godaddy, deploy, haproxy, nginx-proxy, network, postgres, proxmox, server, ssl, unifi, vm-decommission, website-deploy, domain-audit, domain-diagnose, mobaxterm-sessions, fleet-audit |
| **GIS** | geojson-inject, gis-geometry, gis-outlier-detect, gis-spatial-tag, gpkg-export, gpkg-inject, csv-inject, postcodes-io |
| **Media** | elevenlabs, audio-check, video-convert, html-to-video, fal-lipsync, voice-to-text, remotion-presentation, youtube-capture |
| **Design** | diagram-generator, figma-design, figma-to-code, frontend-design |
| **Companies House** | companies-house, companies-house-cs-preflight, companies-house-cs-postfiling, companies-house-status |
| **Comms** | email-send |
| **Utilities** | skill-creator, task-creator, sync-workspace, sync-repos, refresh-settings, repo-init, integrate-repo, check-integrations, enrich-twin, session-close, webapp-creator, submission-reset, snagit, watlas, cityfibre-bitbucket-refresh, mept-fixtures-prep, pc-logs, pc-host-canary |
| **Banking** | barclays-inbox |
| **Booking** | acuity |
| **External** | respond-io-ingest, respond-io-media-capture, web-capture-site, hubspot-academy-extract |
| **UK** | uk-vehicles |

## Development Conventions

- Use conventional commit messages (`feat:`, `fix:`, `docs:`, `refactor:`)
- Keep PRs focused and small
- Follow the established directory structure
- No secrets in git — credentials live in `~/repos/tumai-hq/.mindatlas/credentials/`

## Related Repositories

### Brain Tier (tumai-hq)

| Repository | Purpose |
|------------|---------|
| `mind-atlas` | This repo - HEAD, research, MIND Jira planning home |
| `skills` | Shared AI skills (121) |
| `business-hub` | Business operations |
| `family-hub` | Personal/family life management |
| `beauty-hub` | Kseniia's beauty business |
| `marketing-hub` | Marketing operations |
| `learning-hub` | Education and training |
| `it-hub` | IT infrastructure and operations |
| `sql-hub` | Central SQL workspace |
| `integrations-hub` | Vendor layer - per-vendor docs, vault contracts, smoke tests, tested adapters; cut-off register (Jira: INTEG) |

### Product Tier (tumai-products)

| Repository | Purpose |
|------------|---------|
| `engine-core` | Reusable portal foundation (Go + Vue/DevExtreme) |
| `node-agent` | Infrastructure monitoring agent (Go + Vue) |
| `my-first-app` | Sandbox web app (Vue + Go) |
| `buildsmart` | Product (Jira: BSMART) - CityFibre FTTH DepoNet dataset custody + viewer |
| `file-sync` | Product (Jira: FSYNC) |
| `finance-portal` | Product |
| `mail-atlas` | Product |
| `media-capture` | Product |
| `media-forge` | Product (Jira: MFORGE) |
| `web-capture` | Product (Jira: WCAP) |
| `screen-capture` | Product (Jira: SCAP) |
| `codex` | Product (Jira: CODEX) |
| `work-atlas` | Product (Jira: WATLAS) |
| `buildsmart-portal` | Buildsmart portal - DepoNet dataset custody, S3-to-UNAS transfer, viewer (Jira: BSMART) |
| `front-desk` | Front Desk (internal name) - agent-first multi-channel product: one AI agent per business on every channel, people behind the desk who supervise and take over; Oblique Beauty first tenant (Jira: FDESK) |

### Programme Tier (tumai-programmes)

| Repository | Purpose |
|------------|---------|
| `kseniia` | Kseniia Brow Art programme |
| `kseniia-website` | Renewed kseniia.co.uk (Nuxt + Tailwind, replacing Tilda) |
| `kseniia-website-design` | Kseniia website design assets |
| `kseniia-academy` | Kseniia Academy programme (strategy + content) |
| `kseniia-academy-webapp` | Academy webapp (Nuxt + Go) |
| `kseniia-academy-webapp-design` | Academy webapp design assets |
| `kseniia-portal-api` | Kseniia portal API |
| `kseniia-portal-web` | Kseniia portal web frontend |
| `kseniia-portal-design` | Kseniia portal design assets |
| `kseniia-client-web` | Kseniia client-facing web app (lightweight, no DevExtreme) |
| `limitless` | Limitless programme |
| `limitless-portal` | Limitless portal webapp |
| `limitless-portal-design` (this repo) | Limitless portal design assets |
| `limitless-website` | Limitless public website (Nuxt SSG) |
| `stationroadclinic-co-uk` | Station Road Clinic programme |
| `stationroadclinic-co-uk-portal` | Clinic patient portal |
| `stationroadclinic-co-uk-website` | Clinic public website |
| `vasilyev-co-uk-website` | vasilyev.co.uk website |
| `tumai-co-uk` | tumai.co.uk website (Tumai Management Ltd corporate site - Tilda to Nuxt migration, Jira: TUMUK) |
| `tumaifibre` | Tumai Fibre programme + tumaifibre.co.uk website (single-repo monorepo, Jira: TFIBRE) |
| `ftth-acquisition-dd-framework` | Tumai Fibre Scope of Work framework (vendor-neutral, mined from CF/Cheetah portfolio; TFIBRE workstream) |
| `cf-condor-migration` | CityFibre Condor FTTH migration |
| `cf-migration-genoa` | CityFibre Genoa FTTH pre-migration DD |
| `cf-migration-falcon` | CityFibre Falcon FTTH pre-migration DD |
| `cf-migration-osprey` | CityFibre Osprey FTTH pre-migration DD |
| `cf-migration-cougar` | CityFibre Cougar FTTH migration |
| `cf-migration-engine` | CityFibre MigrationEngine programme |
| `cf-migration-reference` | Shared CityFibre migration reference assets |
| `cf-migration-template` | CityFibre migration project template |
| `cf-comarch-reference` | CityFibre Comarch reference assets |
| `cf-me-performance-testing-strategy` | CityFibre MigrationEngine performance testing strategy (Tumai deliverable) |
| `cf-me-test` | ME-TEST online testsuite-driver service for the CityFibre IME migration engine pipeline |
| `cf-test-wrapper` | TEST-WRAPPER testsuite-driver running CityFibre migration jobs through Purplecube (`-549` / `-deploy549` are version/deploy variants) |
| `cf-ime-validation` | CF IME Validation (Hotspots) programme (Jira: CFIMEV) |
| `ime-hotspots` | IME hotspots analysis |
| `ime-mock` | IME mock service (OpenAPI codegen + HTTP server; `-mx480` is a variant) |
| `depotnet` | Depotnet programme (Jira: BSMART) - legacy DepoNet/VST data rescue |
| `cf-platform-reference` | CityFibre engineering platform reference - GitHub org, CI/CD, EKS/Argo CD GitOps, ECR, Secrets Manager, RDS, SSO, guardrails; shared truth for every Tumai app inside CityFibre (Jira: CFPLAT) |
| `kseniia-portal-app` | Kseniia Portal App - practitioner phone edition of portal.kseniia.co.uk (native Android + iOS, Jira: KPAPP) |
| `kseniia-portal-app-design` | Kseniia Portal App design assets - Figma exports, design notes, source screenshots (Jira: KPAPP) |
| `oblique` | Oblique Beauty programme - AI booking concierge pilot over Phorest for three South Kensington salons, Telegram first then mobile web; catalogue-as-data thesis (Jira: OBLQ) |

### Integrations Tier (tumai-integrations)

Runnable integration workloads - `<vendor>-sandbox` prototypes per vendor and `<vendor>-hub` / `<vendor>-gateway` shared services (org created 2026-08-31, DEC-INTEG-006, registered MIND-100). Boundary rule (DEC-INTEG-006, extends DEC-INTEG-004): vendor knowledge, vault contracts, adapters and smoke tests stay in `tumai-hq/integrations-hub`; runnable prototypes and shared gateway services live in `tumai-integrations`; a prototype graduating into a product moves to `tumai-products` deliberately, never by drift. Naming (vendor-first, decided 2026-09-01, DEC-INTEG-006 amendment): `<vendor>-<role>[-<qualifier>]` - `<vendor>` = the integrations-hub slug; `<role>` = `sandbox` (may rot and be archived; topic `sandbox`), `tenant` (registration/config repo a platform reads), `hub` / `gateway` (production treatment). First repo: `telegram-hub` (INTEG-38); next: `argo-cd-sandbox`, `argo-cd-tenant`, `argo-cd-sandbox-espresso` (Tumai twin platform).

| Repository | Purpose |
|------------|---------|
| `telegram-hub` | Telegram bot-host - multi-bot registry, plugin handlers, admin UI; long-poll v1 (Jira: INTEG, Epic INTEG-36) |
| `argo-cd-sandbox` | Tumai twin platform - Argo CD, ApplicationSet, Gateway API, ESO, admission policy, kind config (Jira: INTEG-43) |
| `argo-cd-tenant` | Tumai twin platform - Argo CD registrations apps/<app>/<env>/<tenant>/config.yaml (Jira: INTEG-43) |
| `argo-cd-sandbox-espresso` | Tumai twin platform - espresso sample app, our copy of CityFibre's cf-k8s-espresso-deployment shape (Jira: INTEG-43) |

### Tumai CF Platform (tumai-cf-platform)

The Tumai CF platform - a one-to-one replica of CityFibre's engineering platform org (`cityfibre-enterprise-architects`) on Tumai's side, built to rehearse and deliver IME-HOTSPOTS and later Tumai apps for CityFibre with full visibility (org created 2026-09-07, CFPLAT DEC-004, registered MIND-105; Jira: CFPLAT, epic CFPLAT-8; plan `cf-platform-reference/platform/tumai-cf-platform/README.md`). The org policy mirrors CityFibre's (Actions allow-list, read-only workflow token, owner-only repo creation, the same org secret `CICD_PRIVATE_KEY` and variable `CICD_APP_ID`). Naming: CityFibre's exact repo names, not the vendor-first INTEG convention - `ime-hotspots` and `cf-k8s-espresso-deployment` (app deployment repos), `tumai-eks-tenant` (Argo CD registrations, tenant `tumai`), `platform` (IaC, add-ons, runbook, journal). Nothing on screen says twin, sandbox or mirror. The reusable installer kit stays in `tumai-integrations/argo-cd-sandbox` (INTEG-43).

| Repository | Purpose |
|------------|---------|
| `tumai-eks-tenant` | Argo CD registrations for the Tumai CF platform - apps/<app>/<env>/tumai/config.yaml, CityFibre's cf-enterprise-architects-eks-tenant shape (Jira: CFPLAT) |
| `cf-k8s-espresso-deployment` | Espresso sample app for the Tumai CF platform - Go service, Dockerfile, kustomize base and overlays, rendered-branch workflows; the shape of CityFibre's cf-k8s-espresso-deployment (Jira: CFPLAT) |
| `platform` | The Tumai CF platform itself - Argo CD project + ApplicationSet over tumai-eks-tenant, gateway, External Secrets, admission policy, namespace limits, cluster IaC, runbook and journal; the platform-team side CityFibre does not show us (Jira: CFPLAT) |
| `me-runner` | ME-RUNNER (Migration Engine Runner) deployment repo on the Tumai CF platform - the primary line of ME-TEST for CityFibre's Kubernetes platform: code, Dockerfile, kustomize base/overlays, rendered-branch workflows; rehearsal of cityfibre-enterprise-architects/me-runner (Jira: CFPLAT E8, app side CFMEPT Track T) |

### Legacy (pre-Claude Code workflow)

- `fedorgithub/cheetah-document-analysis` - Cheetah FTTH programme. Migration to `tumai-programmes` pending; delivery takes priority.
- `fedorgithub/cf-cheetah-migration-sql` - already a separate repo.
- `qoobo`, `tumaicoding` - to be reviewed over time.

> **Note:** `web-portal` and `frontend-shared` have been retired. See `it-hub` for frontend patterns and component references.

---

*Created 2026-03-05. Part of the [tumai-programmes](https://github.com/tumai-programmes) ecosystem.*
