# Wizard Flows v2 — Greg's Original Text

> **Purpose:** Re-align wizard content with Greg's original audit instruments and amendments
> **Source of truth:** `limitless/methodology/invincibility-blueprint/toolkit/`
> **Previous version:** `wizard/` (v1 — our processed/adapted version, preserved in place)
> **Portal:** [limitless-portal](https://github.com/tumai-programmes/limitless-portal) (code)
> **Programme:** [limitless](https://github.com/tumai-programmes/limitless) (methodology)

---

## Why v2?

Greg reviewed the live portal and identified that the questionnaire text and descriptions differ from his original instruments. This version restores Greg's exact text as the canonical content for all three audit wizards.

### Changes from v1

| Area | What changed |
|------|-------------|
| **All instruments** | Content restored to Greg's original text — identical copy, no paraphrasing |
| **01 Company Audit, Section D3** | OD Stage descriptions updated per Greg's amendment (`ch01-changes-required.md`) |

### Amendment applied

Per `ch01-changes-required.md`, the D3 OD Stage Self-Assessment descriptions were updated to use simpler terminology:

| Stage | v1 (old) | v2 (new) |
|-------|----------|----------|
| 1 | Start-up ("Heroic effort") | Heroic effort (Start-up) |
| 2 | Regular Management ("Rational discipline") | Managed delivery |
| 3 | Viral Expansion ("Competitive growth") | Scalable growth |
| 4 | Synergistic Brotherhood ("Mission-driven collaboration") | Aligned performance |

---

## Structure

```
wizard-v2/
├── README.md                          ← You are here
├── 01-company-audit.md                ← Greg's original text + D3 amendment
├── 02-manager-audit.md                ← Greg's original text (identical)
└── 03-engineer-mini-audit.md          ← Greg's original text (identical)
```

## Workflow

1. Review each file against the live portal
2. Identify all text differences between v2 (Greg's original) and v1 (current portal)
3. Update the frontend instrument definitions in `limitless-portal` to match v2
4. Test on dev environment
5. Deploy to production

---

*Created 9 March 2026. Restores Greg's original instrument text as the canonical wizard content.*
