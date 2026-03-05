# CLAUDE.md — Limitless Portal Design

Primary instructions and context for Claude when working in this repository.

## Project Overview

**Limitless Portal Design** is the product planning, UI design, architecture specification, and roadmap repository for the Limitless Modus Portal — an AI-powered operational transformation SaaS platform. This repo separates product design concerns from both the business programme (`limitless`) and the code implementation (`limitless-portal`).

**Repository:** https://github.com/tumai-programmes/limitless-portal-design
**Organisation:** tumai-programmes (Programme Tier)
**Code repo:** `../limitless-portal/` — [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal) (Go + Vue implementation)
**Programme repo:** `../limitless/` — [tumai-programmes/limitless](https://github.com/tumai-programmes/limitless) (business strategy, methodology, engagements)
**SQL operations:** [tumai-hq/sql-hub/limitless-portal](https://github.com/tumai-hq/sql-hub) (monitoring, diagnostics, reports)
**Live URL:** https://nano.limitlessmodus.com/

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
| Skills (58) | `../../tumai-hq/skills/` — [tumai-hq/skills](https://github.com/tumai-hq/skills) |
| API Credentials | `~/.mindatlas/credentials/.env` (local only) |
| Shared Config | `~/.mindatlas/config/` |
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
| **Documents** | pdf, docx, pptx, pptx-to-pdf, xlsx, book-to-markdown, note-to-pdf, note-to-word |
| **Research** | article-reflection, source-digest |
| **Finance** | xero, airtable, rent-invoice, rent-invoice-approve |
| **Infrastructure** | cloudflare, statuscake, deploy, haproxy, network, postgres, proxmox, server, ssl, unifi, vm-decommission |
| **Media** | elevenlabs, fal-lipsync, remotion-presentation, video-convert, voice-to-text, html-to-video |
| **Google** | gmail, gcalendar, gdocs, gsheets, gdrive-sync, youtube, youtube-notes |
| **Atlassian** | atlassian-goals, atlassian-projects, confluence-sync, confluence-scan, confluence-summarise, confluence-publisher, confluence-manager, confluence-pull |
| **Utilities** | diagram-generator, export-images, skill-creator, task-creator, sync-workspace, sync-repos, refresh-settings, integrate-repo |
| **Development** | webapp-creator, website-deploy |
| **System** | check-integrations, enrich-twin |

## Development Conventions

- Use conventional commit messages (`feat:`, `fix:`, `docs:`, `refactor:`)
- Keep PRs focused and small
- Follow the established directory structure
- No secrets in git — credentials live in `~/.mindatlas/credentials/`

## Related Repositories

### Brain Tier (tumai-hq)

| Repository | Purpose |
|------------|---------|
| `mind-atlas` | HEAD — Research, orchestration |
| `skills` | Shared AI skills (58) |
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
