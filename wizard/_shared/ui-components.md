# Shared UI Components & Patterns

> **Status:** Living reference — updated as shared patterns are built and refined
> **Applies to:** All three wizard flows (01 Company Audit, 02 Manager Audit, 03 Engineer Mini-Audit)

---

## Question Type Components

These Vue components render individual question types across all wizards. Each maps to one entry in the Question Type Taxonomy (see [specification.md](specification.md) §1.3).

| Component | Question Types Rendered | Used In |
|-----------|----------------------|---------|
| `QuestionRenderer.vue` | Dispatcher — routes to specific type components | All wizards |
| `FreeText` | `FreeText`, `FreeTextLong` | All |
| `SingleSelect` | `SingleSelect` (radio group or dropdown) | All |
| `MultiSelect` | `MultiSelect`, `MultiSelectCapped` | 02, 03 |
| `ConsentCheckbox` | `ConsentCheckbox` (gate) | All (Step 0) |
| `ResponseMode` | `ResponseMode` (4-option radio + conditional sub-fields) | 01, 02 |
| `TableGrid` | `TableGrid` (structured editable table) | All |
| `RatingGrid` | `RatingScaleWithEvidence` (rating dropdown + evidence per item) | 01, 02 |
| `FileUpload` | `FileUpload` (drag-and-drop zone) | 01, 03 |
| `ChecklistUpload` | `ChecklistUpload` (checklist with ResponseMode per item) | 01, 02 |
| `NumberInput` | `NumberInput` (numeric with optional unit) | 01, 02 |

### Implementation location

```
limitless-portal/frontend/src/components/questions/
├── QuestionRenderer.vue
└── (individual type components)
```

---

## Wizard Shell Components

Shared structural components that wrap all three wizard flows.

| Component | Purpose | Notes |
|-----------|---------|-------|
| Sidebar navigation | Section list with completion icons (checkmark / current / pending) | Adapts section count per instrument |
| Context banner | Participant name, instrument, engagement, completion % | Persistent below top bar |
| Progress bar | Visual completion indicator | In sidebar footer |
| Bottom navigation | "Back" link + "Save and continue" button | Green CTA on every step |
| Section header | Step counter, title, description, question count, estimated time | Phase transition pattern (HMRC takeaway #10) |
| Auto-save indicator | "Saved" / "Saving..." / "Save failed" | Top bar, right side |

### Implementation location

```
limitless-portal/frontend/src/components/
├── wizard/          (shell components)
└── questions/       (question type components)
```

---

## Shared Patterns

### ResponseMode pattern

Every evidence request uses the same 4-option radio group:

```
○ Upload what we have          → [File drop zone]
○ Provide a link/location      → [URL field]
○ Not available yet            → (no sub-field)
○ Will send later              → [ETA text field]
```

### Conditional visibility (showWhen)

Sections and questions can be conditionally shown based on prior answers. Implemented via `showWhen` in the Pinia wizard store (`frontend/src/stores/wizard.ts`).

Used for:
- **02 Manager Audit** — Section 5 routes by department (Q1.3)
- **03 Engineer Mini-Audit** — Section 5 routes by role type (Q1.2)
- Various conditional questions (e.g., "Other" text field when "Other" is selected)

### Consent gate

Step 0 in all three wizards uses a `ConsentCheckbox` that disables the "Save and continue" button until checked.

---

## Design Tokens

See [../design-tokens/](../design-tokens/) for the full token reference:
- **Colours:** `colours.md`
- **Typography:** `typography.md`
- **Spacing:** `spacing.md`

Key tokens used across all wizards:
- Topbar: `#1E2333` (dark navy)
- Accent: `#1E5AC7` (blue, interactive elements)
- Success/submit: `#1E8C57` (green)
- Background: `#F5F7FA` (light grey)
- Font: Inter (Regular, Medium, Semi Bold, Bold)

---

## Related Files

| File | Purpose |
|------|---------|
| [specification.md](specification.md) | Full functional specification |
| [figma-layouts.md](figma-layouts.md) | Figma wireframe reference |
| [ui-takeaways-hmrc.md](ui-takeaways-hmrc.md) | HMRC-inspired UI patterns |
| [right-panel.md](right-panel.md) | Right panel specification (section guidance, status, help, chat, AI video) |
