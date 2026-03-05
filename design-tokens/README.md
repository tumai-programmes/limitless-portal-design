# Design Tokens — Limitless Modus Portal

This directory is the **source of truth** for the Limitless Modus visual design system. All colours, typography, spacing, border radii, and shadows are defined here in human-readable markdown.

The code implementation of these tokens lives in `limitless-portal/frontend/src/styles/` as SCSS / CSS Custom Properties. When updating the design system, **change the spec here first**, then update the code repo to match.

## Token Files

| File | Contents |
|------|----------|
| `colours.md` | Brand palette, semantic colour roles, status colours, neutral scale |
| `typography.md` | Font family, weight, type scale (sizes + line heights) |
| `spacing.md` | Spacing scale, border radii, shadows, layout constants |

## Naming Convention

All CSS Custom Properties use the `--lm-` prefix (Limitless Modus) to avoid collisions with DevExtreme's `--dx-` variables.

Pattern: `--lm-{category}-{name}`

Examples:
- `--lm-color-primary` — brand primary colour
- `--lm-font-size-body` — base body text size
- `--lm-space-md` — medium spacing unit

## Workflow

```
Design repo (here)          Code repo (limitless-portal)
──────────────────          ────────────────────────────
colours.md          ──→     frontend/src/styles/_variables.scss
typography.md       ──→     frontend/src/styles/_typography.scss
spacing.md          ──→     frontend/src/styles/_variables.scss
```
