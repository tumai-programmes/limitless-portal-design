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
| Skills (115) | `../../tumai-hq/skills/` - [tumai-hq/skills](https://github.com/tumai-hq/skills) |
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
| **Finance** | xero, xero-ap-invoice, rent-invoice, rent-invoice-approve, hmrc, tinytax, tumai-management-payroll-close, invoice-pdf, vat-invoice-request, n8n-ap-registry, edgematics-invoice, edgematics-agreement, cleaning-bill-ref |
| **Airtable** | airtable |
| **Atlassian** | jira, atlassian-discovery, atlassian-goals, atlassian-projects |
| **Confluence** | confluence-sync, confluence-scan, confluence-summarise, confluence-publisher, confluence-manager, confluence-pull, confluence-email-scan |
| **Google** | gmail, gcalendar, gdocs, gsheets, gdrive-sync, gworkspace-admin, google-ads, google-analytics, youtube, youtube-notes |
| **SEO** | bing-webmaster |
| **Infrastructure** | cloudflare, statuscake, godaddy, deploy, haproxy, nginx-proxy, network, postgres, proxmox, server, ssl, unifi, vm-decommission, website-deploy, domain-audit, domain-diagnose, mobaxterm-sessions |
| **GIS** | geojson-inject, gis-geometry, gis-outlier-detect, gis-spatial-tag, gpkg-export, gpkg-inject, csv-inject, postcodes-io |
| **Media** | elevenlabs, audio-check, video-convert, html-to-video, fal-lipsync, voice-to-text, remotion-presentation, youtube-capture |
| **Design** | diagram-generator, figma-design, figma-to-code, frontend-design |
| **Companies House** | companies-house, companies-house-cs-preflight, companies-house-cs-postfiling, companies-house-status |
| **Comms** | email-send |
| **Utilities** | skill-creator, task-creator, sync-workspace, sync-repos, refresh-settings, integrate-repo, check-integrations, enrich-twin, session-close, webapp-creator, submission-reset, snagit, cityfibre-bitbucket-refresh, mept-fixtures-prep |
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
| `mind-atlas` | HEAD — Research, orchestration |
| `skills` | Shared AI skills (115) |
| `business-hub` | Business operations (Tumai group) |
| `family-hub` | Personal/family management |
| `beauty-hub` | Kseniia's beauty business |
| `marketing-hub` | Marketing operations |
| `learning-hub` | Education and training |
| `it-hub` | IT infrastructure and operations |

### Product Tier (tumai-products)

| Repository | Purpose |
|------------|---------|
| `node-agent` | Infrastructure monitoring agent (Go + Vue) |
| `my-first-app` | Sandbox web app (Vue + Go) |

### Programme Tier (tumai-programmes)

| Repository | Purpose |
|------------|---------|
| `kseniia` | Kseniia Brow Art programme |
| `kseniia-website` | kseniia.co.uk (Nuxt + Tailwind) |
| `kseniia-academy` | Kseniia Academy programme (strategy + content) |
| `kseniia-academy-webapp` | Academy webapp (Nuxt + Go) |
| `limitless` | Programme: business strategy, methodology, engagements |
| `limitless-portal-design` | This repo — Design: product design, UI/UX, architecture, style system |
| `limitless-portal` | Code: implementation (AI-written), deployment-ready artefacts |
| `limitless-website` | Limitless public website (Nuxt SSG) at limitlessmodus.com |
| `stationroadclinic-co-uk` | Station Road Clinic programme |
| `stationroadclinic-co-uk-portal` | Clinic patient portal |
| `stationroadclinic-co-uk-website` | Clinic public website |
| `vasilyev-co-uk-website` | vasilyev.co.uk website |
| `cf-condor-migration` | CityFibre Condor FTTH migration |

> **Note:** `web-portal` and `frontend-shared` have been retired. See `it-hub` for frontend patterns and component references.

---

*Created 2026-03-05. Part of the [tumai-programmes](https://github.com/tumai-programmes) ecosystem.*
