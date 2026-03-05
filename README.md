# Limitless Portal — Product Design & Planning

Product planning, UI design, architecture specs, and roadmap for the Limitless Modus Portal.

**Code repo:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal) (Go + Vue implementation)
**Programme repo:** [tumai-programmes/limitless](https://github.com/tumai-programmes/limitless) (business strategy, methodology, engagements)
**SQL operations:** [tumai-hq/sql-hub/limitless-portal](https://github.com/tumai-hq/sql-hub) (monitoring, diagnostics, reports)
**Live URL:** [nano.limitlessmodus.com](https://nano.limitlessmodus.com/)

---

## Repository Structure

| Path | Purpose |
|------|---------|
| `architecture/` | Portal architecture spec, system context, data model, API design, status tracker |
| `wizard/` | Wizard UI specifications — per-instrument layout designs |
| `wizard/images/` | Wizard screenshots, architecture diagrams, HMRC reference (66 images) |
| `deliverables/` | Formatted outputs (PDF, DOCX) of wizard specs and plans |
| `working/` | Session notes and progress updates |
| `config/` | Domain configuration |

---

## Architecture

See [architecture/portal-spec.md](architecture/portal-spec.md) for the full architecture specification including:

- Technology stack (Go + Vue 3 + DevExtreme + Supabase + S3 + Azure SSO)
- System context diagram
- Database strategy and abstraction layer
- File storage strategy
- Data model (Engagements, Submissions, Responses, Uploads, Interviews, Findings)
- API design (Authentication, Engagements, Submissions, Uploads, Interviews, Synthesis)
- User roles and auth flow
- Implementation phases (1–5)
- Design decisions log
- Open questions

## Current Status

See [architecture/status.md](architecture/status.md) for detailed progress including:

- What works (dev-mode auth, Company Audit wizard, deployment pipeline)
- What's scaffolded/partial (Azure SSO, S3 client)
- What's not started (Manager/Engineer audits, interviews, AI synthesis)
- Known issues and action plan
- Deployment status

---

## Wizard UI Specifications

Each audit instrument (01–03) has been designed as a multi-step wizard. Specs include field-by-field layouts with DevExtreme component mapping.

| Spec | Instrument | Status |
|------|-----------|--------|
| [wizard/specification.md](wizard/specification.md) | Overall wizard specification | Complete |
| [wizard/specification-plan.md](wizard/specification-plan.md) | Specification plan and approach | Complete |
| [wizard/01-company-audit-layouts.md](wizard/01-company-audit-layouts.md) | Company Audit (01) — 11 sections, ~200 questions | Complete, implemented |
| [wizard/02-manager-audit-layouts.md](wizard/02-manager-audit-layouts.md) | Manager Audit (02) — 13 sections with dept routing | Complete, not yet implemented |
| [wizard/03-engineer-mini-audit-layouts.md](wizard/03-engineer-mini-audit-layouts.md) | Engineer Mini-Audit (03) — 10 sections with job-type modules | Complete, not yet implemented |
| [wizard/figma-layouts.md](wizard/figma-layouts.md) | Figma design layouts and component mapping | Complete |
| [wizard/ui-takeaways-hmrc.md](wizard/ui-takeaways-hmrc.md) | UI lessons from HMRC reference analysis | Complete |

---

## Images & Reference Materials

Wizard step screenshots, architecture diagrams, and HMRC reference images are all in `wizard/images/` (66 files), referenced directly by the wizard markdown specs.

| Type | Examples |
|------|----------|
| Wizard step screenshots | `wizard-01-step00-consent.png`, `wizard-02-step01-personal.png`, etc. |
| Architecture diagrams | `conditional-routing.png`, `participant-journey.png` |
| HMRC reference screenshots | `image-20250131-*.png` (12 images — wizard UX reference) |

Formatted deliverables (PDF, DOCX) are in `deliverables/`.

---

## Cross-References

### Methodology (Source of Truth)

The wizard specifications implement Greg Kurnikov's diagnostic instruments. The methodology content (what gets asked, why, and how results are interpreted) lives in the programme repo:

- **Instruments:** `limitless/methodology/invincibility-blueprint/toolkit/` (6 instruments: 01–06)
- **Legal framework:** `limitless/methodology/invincibility-blueprint/legal/`
- **Invincibility Blueprint overview:** `limitless/methodology/invincibility-blueprint/README.md`

### Code Implementation

The portal code implementing these designs lives in `limitless-portal`:

- **Go backend:** `limitless-portal/internal/` (handlers, middleware, storage, models)
- **Vue frontend:** `limitless-portal/frontend/src/` (views, components, stores)
- **SQL migrations:** `limitless-portal/migrations/`
- **Deployment:** `limitless-portal/deploy/` (Dockerfile, systemd, CI/CD)

### SQL Operations

Operational queries for the deployed portal live in `sql-hub/limitless-portal/`:

- Monitoring, diagnostics, reports, seed data

---

## Boundary Rules

- **"Is it about what Greg diagnoses?"** — goes in `limitless/methodology/`
- **"Is it about how the portal looks, works, or is architected?"** — goes here (`limitless-portal-design/`)
- **"Is it code, migrations, or deploy config?"** — goes in `limitless-portal/`
- **"Is it a SQL query for monitoring or analysis?"** — goes in `sql-hub/limitless-portal/`

---

*Part of the [Limitless Modus](https://github.com/tumai-programmes/limitless) ecosystem.*
