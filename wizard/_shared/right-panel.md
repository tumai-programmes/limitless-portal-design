# Wizard Right Panel — Specification

> **Status:** Draft — ready for Phase 1 execution
> **Created:** 8 March 2026
> **Affects:** All three wizard flows (01 Company Audit, 02 Manager Audit, 03 Engineer Mini-Audit)
> **Layout change:** Transforms the current two-column wizard (sidebar + content) into a three-column layout (sidebar + scrollable content + fixed right panel)

---

## Overview

Introduce a persistent **right panel** to the shared wizard layout. The main content area retains its vertical scroll; the right panel is **fixed** (does not scroll with content) and provides contextual guidance, status intelligence, and interactive support for the current section.

### Layout Structure (Before → After)

**Before (current):**

```
┌──────────────────────────────────────────────┐
│ TopBar                                        │
├──────────────────────────────────────────────┤
│ ContextBanner                                 │
├──────────┬───────────────────────────────────┤
│ Sidebar  │ Scrollable Content                 │
│          │                                    │
│          │                                    │
├──────────┴───────────────────────────────────┤
│ BottomBar                                     │
└──────────────────────────────────────────────┘
```

**After (proposed):**

```
┌───────────────────────────────────────────────────────┐
│ TopBar                                                 │
├───────────────────────────────────────────────────────┤
│ ContextBanner                                          │
├──────────┬──────────────────────────┬─────────────────┤
│ Sidebar  │ Scrollable Content       │ Right Panel     │
│ ~200px   │ flex: 1                  │ ~320px          │
│          │ (scrolls independently)  │ (fixed, no      │
│          │                          │  scroll with    │
│          │                          │  content)       │
├──────────┴──────────────────────────┴─────────────────┤
│ BottomBar                                              │
└───────────────────────────────────────────────────────┘
```

---

## Right Panel Modules

The right panel is composed of a **section header** followed by stacked module cards, rendered top-to-bottom. Content is **section-aware** — it updates when the participant navigates between wizard steps.

### Header: Section Title (persistent anchor)

The section title is repeated at the top of the right panel as a persistent wayfinding anchor. Once the participant scrolls past the heading in the main content area, the sidebar shows only a short label — this right-panel title keeps the participant oriented without switching focus.

**Design rationale — visual hierarchy:**

The right panel is a *supporting* surface. Its typography must never compete with the main content area for the participant's primary attention. The section title in the right panel uses a deliberately quieter treatment:

| Element | Main content heading | Right panel section title |
|---------|---------------------|--------------------------|
| **Font size** | `--lm-font-size-xl` (20px) | `--lm-font-size-sm` (12px) |
| **Weight** | `--lm-font-weight-semibold` (600) | `--lm-font-weight-semibold` (600) |
| **Transform** | None (sentence case) | `text-transform: uppercase` |
| **Letter spacing** | Normal | `0.08em` (expanded tracking) |
| **Colour** | `--lm-color-text` (#333) | `--lm-color-text-secondary` (#666) |
| **Role** | Primary heading — draws the eye | Eyebrow label — readable when glanced at, invisible when focused on main content |

This "eyebrow label" pattern (small, uppercase, tracked, muted) is a standard approach for secondary navigation anchors. It reads clearly when the participant deliberately looks at the right panel, but its low visual weight avoids pulling attention away from the main form.

| Attribute | Detail |
|-----------|--------|
| **Content** | Matches the section's `label` from the instrument definition (e.g., "A: Organisational Foundation") |
| **Position** | Pinned at the very top of the right panel, above all module cards, with a subtle bottom border (`--lm-color-border-light`) separating it from the modules below |
| **Spacing** | `padding: var(--lm-space-md) var(--lm-space-lg)` — compact, does not take up excessive vertical space |
| **Step indicator** | Optionally prefixed with the step number (e.g., "STEP 2 — A: ORGANISATIONAL FOUNDATION") using the same muted style |

### Module 1: Section Description

A brief contextual description of the current section, explaining its purpose and what the participant should focus on.

| Attribute | Detail |
|-----------|--------|
| **Content source** | Per-section text defined in the instrument definition (new `description` field on `SectionDef`) |
| **Rendering** | 2–4 sentences in `--lm-font-size-sm` (12px), `--lm-color-text-secondary` (#666), `--lm-font-weight-normal` (400). Light card with `--lm-color-bg-page` background |
| **Behaviour** | Updates automatically when the participant navigates to a different section |
| **Fallback** | If no description is defined for a section, this module is hidden |

### Module 2: Section Completion Status

A traffic-light status indicator for the current section, reflecting how complete and valid the participant's answers are.

| Status | Colour | Meaning |
|--------|--------|---------|
| **Not started** | Grey | Section has not been touched — no answers entered |
| **In progress** | Amber/Yellow | Section has been touched but not all required fields are filled |
| **Needs attention** | Red | Section is fully filled but AI validation has flagged issues — answers may be nonsensical, contradictory, too brief, or implausible |
| **Complete** | Green | All required fields filled and no AI flags raised |

| Attribute | Detail |
|-----------|--------|
| **Logic** | Grey → Amber → Red/Green progression, determined by field completeness + optional AI quality check |
| **AI validation** | Background call (debounced, after auto-save) to an AI endpoint that evaluates whether answers are contextually sensible. Not a hard gate — advisory only. Details of the AI validation API to be designed separately |
| **Display** | Coloured badge/pill with status label + short explanation (e.g., "2 of 8 required fields answered" or "AI flagged: answer to A1.1 appears too brief") |
| **Phase note** | Phase 1 delivers Grey/Amber/Green (completeness only). Phase 2 adds Red (AI validation) |

### Module 3: Help & Guidance

A contextual help area providing additional information about the current section's questions, with an option to expand into a detailed overlay.

| Attribute | Detail |
|-----------|--------|
| **Inline content** | 1–3 bullet tips relevant to the current section (e.g., "Tip: Include both formal and informal processes", "Don't worry about perfect formatting — bullet points are fine") |
| **Typography** | Tips in `--lm-font-size-sm` (12px), `--lm-color-text-secondary`, with a small bullet icon or `·` prefix. Card heading "Tips" in `--lm-font-size-sm` (12px), `--lm-font-weight-medium` (500) |
| **Content source** | Per-section tips defined in the instrument definition (new `tips` field on `SectionDef`) |
| **Expand button** | "Learn more" text-link (not a heavy button) — `--lm-font-size-sm`, `--lm-color-secondary`, underline on hover. Opens a DxPopup/modal with a fuller description, what good answers look like, and example responses |
| **Fallback** | If no tips are defined, show a generic message: "Answer as best you can — we will cover gaps in the interview" |

### Module 4: Support Actions

Buttons for the participant to request live support or raise a question.

| Action | Behaviour |
|--------|-----------|
| **Chat with Consultant** | Opens a chat window (or redirects to a chat interface) where the participant can message the engagement consultant (Greg or team). Phase 1: opens a mailto link or simple message form. Phase 2: real-time chat |
| **Submit a Question** | Opens a lightweight form: subject line (pre-filled with section name), message body, optional screenshot attachment. Submits as a support ticket visible to the admin dashboard. Confirmation shown inline |

| Attribute | Detail |
|-----------|--------|
| **Display** | Two stacked buttons (outlined style), placed below the help module |
| **Context** | Both actions auto-tag the current section and question context so the consultant knows where the participant is |
| **Phase note** | Phase 1 delivers "Submit a Question" (form → email to admin). "Chat with Consultant" arrives in Phase 2 |

### Module 5: AI Video Guide

A section-specific short video where a consultant (Greg or Fedor) presents guidance related to the current section. The video uses AI-generated lip-sync over a portrait image with pre-recorded audio.

| Attribute | Detail |
|-----------|--------|
| **Format** | Portrait/vertical video (roughly 9:16 aspect ratio to fit the narrow panel width), 30–90 seconds per section |
| **Production** | Pre-produced per section using the `fal-lipsync` skill (Creatify Aurora): static portrait image + section-specific audio script → lip-synced video |
| **Content** | The consultant addresses the participant directly: "In this section we're looking at [topic]. The key thing is [guidance]. Don't overthink [common concern]..." |
| **Rendering** | Embedded video player within a card, with play button overlay. Auto-play is OFF — participant clicks to play |
| **Fallback** | If no video exists for a section, this module is hidden |
| **Storage** | Videos stored in S3 alongside engagement assets, referenced by instrument + section key |
| **Phase note** | Phase 3 delivery — requires script writing, audio recording, and AI video generation for each section of each instrument |

---

## Data Model Changes

### SectionDef Extension (Instrument Definition)

Add the following optional fields to `SectionDef` in `types/wizard.ts`:

```typescript
interface SectionDef {
  key: string
  label: string
  shortLabel?: string
  showWhen?: ShowWhenCondition

  // Right Panel — new fields
  description?: string          // Module 1: Section description (2-4 sentences)
  tips?: string[]               // Module 3: Help bullet tips
  helpDetail?: string           // Module 3: Expanded help content (markdown)
  videoUrl?: string             // Module 5: AI video guide URL
}
```

### AI Validation Endpoint (Phase 2)

```
POST /api/participant/submissions/{id}/sections/{key}/validate
```

Request: section answers (same as auto-save payload)
Response: `{ status: "ok" | "flagged", flags: [{ questionId, message }] }`

---

## Responsive Behaviour

| Breakpoint | Layout |
|------------|--------|
| **≥ 1280px** | Full three-column layout: sidebar (200px) + content (flex) + right panel (320px) |
| **1024–1279px** | Right panel collapses to a toggle button on the right edge; clicking opens it as an overlay/drawer |
| **< 1024px (tablet)** | Right panel becomes a bottom sheet or top-of-content collapsible section |
| **< 768px (mobile)** | Right panel content available via a floating action button (FAB) → bottom sheet |

---

## Right Panel Typography Summary

All right-panel text is deliberately set smaller and lighter than the main content to maintain visual hierarchy. The main content area is where the participant works; the right panel is where they glance for support.

| Element | Token | Size | Weight | Colour | Notes |
|---------|-------|------|--------|--------|-------|
| Section title | `--lm-font-size-sm` | 12px | 600 | `--lm-color-text-secondary` | Uppercase, tracked 0.08em |
| Module card heading | `--lm-font-size-sm` | 12px | 500 | `--lm-color-text-secondary` | e.g., "Tips", "Status" |
| Module body text | `--lm-font-size-sm` | 12px | 400 | `--lm-color-text-secondary` | Description, tips, status explanation |
| Status badge label | `--lm-font-size-xs` | 11px | 600 | Semantic colour (white on pill) | e.g., "IN PROGRESS" |
| Buttons (support) | `--lm-font-size-sm` | 12px | 500 | `--lm-color-secondary` | Outlined, compact padding |
| "Learn more" link | `--lm-font-size-sm` | 12px | 400 | `--lm-color-secondary` | Underline on hover |

**Contrast test:** At 12px/400/`#666` on `#f8f9fb` background, the WCAG AA contrast ratio is ~5.5:1 — passes. The panel text is readable when looked at directly, but does not pull focus from the 14–20px main content.

---

## Phased Delivery

| Phase | Scope | Dependencies |
|-------|-------|-------------|
| **Phase 1** | Layout shell (three-column), section title header, Module 1 (description), Module 2 (grey/amber/green completeness only), Module 3 (tips + learn-more modal), Module 4 ("Submit a Question" form only) | Instrument definition updates, WizardLayout.vue refactor, new WizardRightPanel.vue component |
| **Phase 2** | Module 2 AI validation (red status), Module 4 "Chat with Consultant" (real-time), AI validation endpoint | Backend API, chat infrastructure |
| **Phase 3** | Module 5 (AI video guides) — script writing, recording, video generation per section per instrument | Content production pipeline, S3 video hosting, fal-lipsync skill |

---

## Phase 1 Execution Plan

### Step 1: Extend TypeScript types

**File:** `limitless-portal/frontend/src/types/wizard.ts`

Add optional right-panel fields to `SectionDef`:

```typescript
interface SectionDef {
  key: string
  label: string
  shortLabel?: string
  showWhen?: ShowWhenCondition

  // Right panel content (all optional)
  description?: string        // Module 1: section purpose (2-4 sentences)
  tips?: string[]             // Module 3: inline help bullets
  helpDetail?: string         // Module 3: expanded help (markdown)
  videoUrl?: string           // Module 5: AI video guide URL (Phase 3)
}
```

No breaking change — all new fields are optional.

### Step 2: Create WizardRightPanel.vue

**File:** `limitless-portal/frontend/src/components/wizard/WizardRightPanel.vue`

New component. Receives props and renders the stacked modules:

| Prop | Type | Source |
|------|------|--------|
| `sectionLabel` | `string` | Current section label (for title header) |
| `stepIndex` | `number` | Current step number |
| `totalSteps` | `number` | Total step count |
| `description` | `string \| undefined` | From instrument definition |
| `tips` | `string[] \| undefined` | From instrument definition |
| `helpDetail` | `string \| undefined` | For "learn more" modal |
| `status` | `'not_started' \| 'in_progress' \| 'complete'` | Computed from answer completeness |
| `requiredCount` | `number` | Total required fields in section |
| `answeredCount` | `number` | Answered required fields |
| `videoUrl` | `string \| undefined` | Phase 3 — hidden if undefined |

Internal structure:
- Section title (eyebrow label, always visible)
- Module 1 card (description) — hidden if no `description`
- Module 2 card (status badge + progress text)
- Module 3 card (tips + "learn more" link) — generic fallback if no `tips`
- Module 4 card ("Submit a Question" button)
- Module 5 card (video) — hidden until Phase 3

### Step 3: Create SubmitQuestionModal.vue

**File:** `limitless-portal/frontend/src/components/wizard/SubmitQuestionModal.vue`

Lightweight DxPopup form:
- Subject (pre-filled: "Question about [Section Label]")
- Message body (DxTextArea)
- Optional attachment (DxFileUploader, single file)
- Submit button → `POST /api/participant/submissions/{id}/questions`
- Success confirmation inline

### Step 4: Refactor WizardLayout.vue

**File:** `limitless-portal/frontend/src/layouts/WizardLayout.vue`

Changes to existing layout:

1. Add `WizardRightPanel` as third child of `.wizard-body`
2. Add new props to pass right-panel data through
3. Update CSS:

```scss
.wizard-body {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.wizard-content {
  flex: 1;
  min-width: 0;           // prevent flex overflow
  overflow: hidden;
  background: var(--lm-color-bg-surface);
}

.wizard-right-panel {
  width: 320px;
  flex-shrink: 0;
  border-left: 1px solid var(--lm-color-border-light);
  background: var(--lm-color-bg-page);
  overflow-y: auto;       // panel scrolls independently if modules overflow
  padding: 0;
}

// Responsive: collapse panel below 1280px
@media (max-width: 1279px) {
  .wizard-right-panel {
    display: none;         // Phase 1: hide. Phase 1b: toggle drawer
  }
}
```

### Step 5: Expose right-panel computed state in wizard store

**File:** `limitless-portal/frontend/src/stores/wizard.ts`

Add computed properties:

```typescript
// Current section's right-panel data
currentSectionLabel    // from sections[currentSectionIndex].label
currentDescription     // from sections[currentSectionIndex].description
currentTips            // from sections[currentSectionIndex].tips
currentHelpDetail      // from sections[currentSectionIndex].helpDetail
currentVideoUrl        // from sections[currentSectionIndex].videoUrl
currentSectionStatus   // computed: 'not_started' | 'in_progress' | 'complete'
currentRequiredCount   // count of required questions in current section
currentAnsweredCount   // count of answered required questions
```

### Step 6: Add per-section content to instrument definitions

**Files:**
- `limitless-portal/frontend/src/instruments/company-audit.ts`
- (Later: `manager-audit.ts`, `engineer-mini-audit.ts`)

Add `description` and `tips` to each section object. Start with Company Audit only (01) — other instruments follow the same pattern once content is written.

Example for Section A:

```typescript
{
  key: 'section_a',
  label: 'A: Organisational Foundation',
  description: 'This section captures how your organisation defines its purpose, values, operating model, and decision-making authority. We are looking for what exists today — not aspirations.',
  tips: [
    'Include both formal documents and informal practices',
    'Bullet points are fine — we will explore detail in interviews',
    'If something doesn\'t exist yet, say so — that\'s useful data'
  ],
  helpDetail: '### What good answers look like\n\n...',
  questions: [...]
}
```

### Step 7: Wire it all together in CompanyAuditWizard.vue

**File:** `limitless-portal/frontend/src/views/CompanyAuditWizard.vue`

Pass right-panel props from wizard store through to `WizardLayout`, which passes them to `WizardRightPanel`.

### Step 8: Backend — question submission endpoint

**File:** `limitless-portal/internal/handlers/participant.go` (or new file)

```
POST /api/participant/submissions/{id}/questions
```

Body: `{ section_key, subject, message, attachment_url? }`
Action: Store in DB + send email notification to engagement admin
Response: `{ id, created_at }`

---

## Execution Checklist

| # | Task | File(s) | Est. |
|---|------|---------|------|
| 1 | Extend `SectionDef` type | `types/wizard.ts` | 5 min |
| 2 | Create `WizardRightPanel.vue` | New file in `components/wizard/` | 45 min |
| 3 | Create `SubmitQuestionModal.vue` | New file in `components/wizard/` | 30 min |
| 4 | Refactor `WizardLayout.vue` — add third column + responsive CSS | `layouts/WizardLayout.vue` | 30 min |
| 5 | Add right-panel computed properties to wizard store | `stores/wizard.ts` | 20 min |
| 6 | Add `description` + `tips` to Company Audit sections | `instruments/company-audit.ts` | 30 min |
| 7 | Wire props through `CompanyAuditWizard.vue` → layout → panel | `views/CompanyAuditWizard.vue` | 15 min |
| 8 | Backend: question submission endpoint | `internal/handlers/` | 30 min |
| | **Total Phase 1** | | **~3.5 hrs** |

---

## Open Questions

1. **Panel width** — 320px is proposed. Should it be narrower (280px) on smaller screens or always 320px until collapse?
2. **Video aspect ratio** — Portrait (9:16) fits the narrow panel. Confirm this is acceptable vs. landscape with letterboxing
3. **AI validation scope** — Should validation run on every auto-save, or only when the participant explicitly clicks "check my answers"?
4. **Chat infrastructure** — What chat platform for Phase 2? Options: embedded widget (Intercom/Crisp), custom WebSocket chat, or simple threaded messaging within the portal
5. **Content authoring** — Who writes the per-section descriptions and tips? Greg (domain expert) or Fedor (with Greg's review)?

---

*Part of the [Wizard Flows](..) — [Limitless Portal Design](../../README.md)*
