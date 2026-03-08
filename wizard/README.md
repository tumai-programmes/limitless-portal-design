# Wizard Flows — Navigator

> **Purpose:** Plan, track, and record the design and testing of all three Invincibility Blueprint self-submission audit wizards
> **Portal:** [limitless-portal](https://github.com/tumai-programmes/limitless-portal) (code)
> **Programme:** [limitless](https://github.com/tumai-programmes/limitless) (methodology)
> **Design source:** Figma project "Limitless Modus Portal" → Page "Wizard — Audit Self-submission"

---

## Status Dashboard

| Flow | Spec | Figma | Frontend | Testing | Issues |
|------|:----:|:-----:|:--------:|:-------:|:------:|
| [01 Company Audit](01-company-audit/) | Done | Done | Implemented | Not started | [0 open](01-company-audit/issues.md) |
| [02 Manager Audit](02-manager-audit/) | Done | Done | Implemented | Not started | [0 open](02-manager-audit/issues.md) |
| [03 Engineer Mini-Audit](03-engineer-mini-audit/) | Done | Done | Implemented | Not started | [0 open](03-engineer-mini-audit/issues.md) |

---

## Structure

```
wizard/
├── README.md                          ← You are here
├── specification-plan.md              ← Specification scope document
│
├── _shared/                           ← Cross-wizard concerns
│   ├── specification.md               ← Master specification (all 3 instruments)
│   ├── figma-layouts.md               ← Figma wireframe overview (54 frames)
│   ├── ui-takeaways-hmrc.md           ← 12 UI patterns from HMRC wizard
│   ├── ui-components.md               ← Shared Vue components & design tokens
│   └── right-panel.md                 ← Right panel spec (guidance, status, help, chat, video)
│
├── 01-company-audit/                  ← Company Audit (Director)
│   ├── README.md                      ← Flow overview & status
│   ├── layouts.md                     ← Per-step Figma reference
│   ├── testing-log.md                 ← Step-by-step test results
│   └── issues.md                      ← Open bugs / improvements
│
├── 02-manager-audit/                  ← Manager Audit (Managers)
│   ├── README.md
│   ├── layouts.md
│   ├── testing-log.md
│   └── issues.md
│
├── 03-engineer-mini-audit/            ← Engineer Mini-Audit (Field engineers)
│   ├── README.md
│   ├── layouts.md
│   ├── testing-log.md
│   └── issues.md
│
└── images/                            ← Shared screenshots & wireframes
```

---

## Quick Links

### Per-Flow

| Flow | Overview | Layouts | Testing | Issues |
|------|----------|---------|---------|--------|
| 01 Company Audit | [README](01-company-audit/README.md) | [layouts](01-company-audit/layouts.md) | [testing-log](01-company-audit/testing-log.md) | [issues](01-company-audit/issues.md) |
| 02 Manager Audit | [README](02-manager-audit/README.md) | [layouts](02-manager-audit/layouts.md) | [testing-log](02-manager-audit/testing-log.md) | [issues](02-manager-audit/issues.md) |
| 03 Engineer Mini-Audit | [README](03-engineer-mini-audit/README.md) | [layouts](03-engineer-mini-audit/layouts.md) | [testing-log](03-engineer-mini-audit/testing-log.md) | [issues](03-engineer-mini-audit/issues.md) |

### Shared

| Document | Purpose |
|----------|---------|
| [Specification](_shared/specification.md) | Full functional spec — all instruments, data model, API |
| [Figma Layouts](_shared/figma-layouts.md) | 54 wireframe overview with node IDs |
| [HMRC Takeaways](_shared/ui-takeaways-hmrc.md) | 12 proven UI patterns |
| [UI Components](_shared/ui-components.md) | Shared Vue components & design tokens |
| [Right Panel](_shared/right-panel.md) | Right panel — section guidance, status, help, chat, AI video |

---

## Test Accounts (CREDO GROUP UK)

| # | Email | Name | Group | Audit | Google Password |
|---|-------|------|-------|-------|-----------------|
| 12 | mark.director@credo-group.co.uk | Mark Director | leadership | 01-company-audit | CdqXDZSKDVAr2qG |
| 13 | adam.finance@credo-group.co.uk | Adam Finance | finance | 01-company-audit | CdqXDZSKDVAr2qG |
| 14 | kevin.operations@credo-group.co.uk | Kevin Operations | operations | 02-manager-audit | CdqXDZSKDVAr2qG |
| 15 | damon.operations@credo-group.co.uk | Damon Operations | operations | 02-manager-audit | CdqXDZSKDVAr2qG |
| 16 | nicholas.field@credo-group.co.uk | Nicholas Field | engineering | 03-engineer-mini-audit | CdqXDZSKDVAr2qG |
| 17 | mathew.field@credo-group.co.uk | Mathew Field | engineering | 03-engineer-mini-audit | CdqXDZSKDVAr2qG |

**Portal URL:** https://portal.limitlessmodus.com

---

## Workflow

1. **Testing** — Open a wizard via the participant dashboard, walk through each step
2. **Log results** — Record pass/fail in the flow's `testing-log.md`
3. **File issues** — Create entries in the flow's `issues.md` for anything broken
4. **Fix in code** — Implement fixes in `limitless-portal` (code repo)
5. **Re-test** — Verify fixes and update the testing log
6. **Close issues** — Move resolved issues to the "Resolved" section

---

*Created 5 March 2026 as part of wizard flow testing preparation.*
