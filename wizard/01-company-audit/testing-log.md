# 01 Company Audit — Testing Log

> **Purpose:** Step-by-step UI testing record for the Company Audit wizard flow
> **Live URL:** https://portal.limitlessmodus.com/company-audit?submission_id=...
> **Code:** `limitless-portal/frontend/src/views/CompanyAuditWizard.vue`
> **Instrument:** `limitless-portal/frontend/src/instruments/company-audit.ts`

---

## Handoff Briefing — 5 March 2026

### Context

The Company Audit wizard is the largest of the three instruments (14 steps, ~160 questions). The frontend instrument definition and wizard view are implemented. This testing pass will walk through every step, checking that:

1. All questions render correctly for their type
2. Navigation (back/forward, sidebar) works
3. Auto-save persists answers
4. Required/optional labels display correctly
5. The consent gate blocks progression until checked
6. The review step shows all answers with edit links
7. Submission locks the wizard

### What is implemented

- **Frontend wizard shell:** Sidebar navigation, context banner, progress bar, bottom nav
- **Question renderer:** Dispatches to type-specific components based on instrument definition
- **Instrument definition:** `company-audit.ts` — all 14 steps with full question inventories
- **Backend endpoints:** `/api/participant/submissions/{id}` (GET), section save (PUT), submit (POST)
- **Auth:** Microsoft Entra ID SSO + Google OAuth2; participant matched by email
- **Dev-mode auth bypass:** JWT_SECRET starting with `dev-` enables local testing without SSO

### Test accounts (CREDO GROUP UK)

| # | Email | Name | Group | Assigned Audit |
|---|-------|------|-------|----------------|
| 12 | mark.director@credo-group.co.uk | Mark Director | leadership | 01-company-audit |
| 13 | adam.finance@credo-group.co.uk | Adam Finance | finance | 01-company-audit |

Google password for all test accounts: `CdqXDZSKDVAr2qG`

### How to test

1. Open https://portal.limitlessmodus.com
2. Sign in with a test account (Google SSO)
3. You should land on the participant dashboard showing the assigned "Company Audit"
4. Click "Start" or "Continue" to enter the wizard
5. Walk through each step, checking rendering and interactions
6. Log results below

### Known pre-existing issues

- Some question types may not have fully styled components yet (RatingScaleWithEvidence, ChecklistUpload are the most complex)
- TableGrid may need horizontal scroll on narrower screens
- Image paths in Figma layouts reference the `images/` folder at `wizard/images/`

---

## Test Sessions

### Session 1 — 8 March 2026 (Code Review)

**Tester:** Claude (static code review — no browser automation available)
**Method:** Full code review of instrument definition, all 14 question components, wizard store, wizard layout, and type definitions
**Account:** N/A (code-level review only; manual browser verification still needed)

> **Note:** This session was a comprehensive static analysis. A follow-up manual browser session is recommended to confirm rendering and interaction in the live environment.

#### Step 0: Consent

| Check | Result | Notes |
|-------|--------|-------|
| Confidentiality statement displays | PASS (code) | `id: '0.1'`, `displayOnly: true`, `displayContent` with `<h3>Confidentiality</h3>` renders via `QuestionRenderer` `.question-display` |
| Upload guidance displays | PASS (code) | `id: '0.2'`, `displayOnly: true` with upload guidance HTML |
| "Before You Start" guidance displays | PASS (code) | `id: '0.3'`, `displayOnly: true` with estimated 60–90 minutes text |
| Consent checkbox renders | PASS (code) | `id: '0.4'`, `type: 'ConsentCheckbox'`, `consentText` populated. `QConsentCheckbox.vue` renders native `<input type="checkbox">` + label |
| "Save and continue" disabled until checked | **FAIL** | **BUG-01:** Button is never disabled. `canGoNext` only checks position (`currentSectionIndex < sections.length - 1`), not consent state. `handleNext()` silently returns if consent unchecked — no visual feedback |
| Checking consent enables button | N/A | Button is always enabled (see BUG-01) |
| Clicking "Save and continue" advances to Step 1 | PASS (code) | After checking consent, `handleNext()` calls `giveConsent()` then `goNext()` |

#### Step 1: Company Context

| Check | Result | Notes |
|-------|--------|-------|
| Section header shows "Step 1 of 13" | **NOTE** | Header shows `label` ("Company Context") but no "Step X of 13" counter in the section header. The sidebar shows position |
| Q1.1 FreeText renders | PASS (code) | `type: 'FreeText'` maps to `QFreeText.vue` → `DxTextBox` |
| Q1.2 MultiSelect renders (4 options) | PASS (code) | 4 options defined; `QMultiSelect.vue` checkbox group. No guard for missing `options` prop (low risk — defined in instrument) |
| Q1.3–1.5 FreeTextLong render | PASS (code) | All `type: 'FreeTextLong'` → `DxTextArea` with auto-resize |
| Q1.6 TableGrid renders (4 rows) | PASS (code) | 2 columns × 4 rows. `QTableGrid.vue` uses `DxTextBox` per cell. No horizontal scroll handling documented |
| Q1.7 FreeTextLong renders | PASS (code) | Standard FreeTextLong |
| Q1.8 ResponseMode renders (4 options) | PARTIAL | `QResponseMode.vue` renders 4 radio options. **BUG-02:** In upload mode, `DxFileUploader` has no `value` or `@value-changed` binding — selected files are never stored in model |
| Q1.9 FreeTextLong + FileUpload renders | **NOTE** | Defined as `type: 'FreeTextLong'` only — no companion FileUpload. Layout spec says "FreeTextLong + FileUpload" but the instrument only uses FreeTextLong. Design–code mismatch (catch-all questions) |
| Required labels shown | PASS (code) | `QuestionRenderer` shows `<span class="question-required">*</span>` when `question.required` |
| Back button disabled (first data step) | PASS (code) | On consent step (index 0) `canGoBack` is false. On step 1 (index 1) `canGoBack` is true — user can go back to consent. Back is disabled only on the consent step itself |
| Auto-save triggers | PASS (code) | `setAnswer()` calls `scheduleAutoSave()` → 2-second debounce → `saveCurrentSection()` |

#### Step 2: A — Org Foundation

| Check | Result | Notes |
|-------|--------|-------|
| 16 questions render | **17 questions** | Instrument has 17 questions (A1.1–A1.4, A2.1–A2.3, A3.1–A3.2, A4.1–A4.7, A.catch). Layout spec says 16. One extra in code |
| 4 sub-group headings visible | **FAIL** | **BUG-03:** No sub-group headings implemented. Questions are listed sequentially without A1/A2/A3/A4 visual grouping. The instrument definition has no sub-group metadata |
| ResponseMode questions show 4-option pattern | PARTIAL | ResponseMode renders but file upload is broken (BUG-02) |
| Catch-all at bottom has FreeTextLong + FileUpload | **NOTE** | `A.catch` is `type: 'FreeTextLong'` only — no FileUpload companion. Same design–code mismatch as Step 1 |

#### Step 3: B — Org Structure

| Check | Result | Notes |
|-------|--------|-------|
| 10 questions render | PASS (code) | 10 questions defined (B1.1–B1.3, B2.1–B2.2, B3.1–B3.2, B4.1, B.catch) |
| ChecklistUpload (B2.2) renders | PARTIAL | `QChecklistUpload.vue` shows checklist items with ResponseMode sub-fields. **BUG-04:** No actual file upload or link input when mode is "upload" or "link" — only `mode` value is stored |
| Display-only guidance (B2.1) | PASS (code) | `displayOnly: true` with `<h4>Regular Duties</h4>` guidance card |
| TableGrid (B4.1) 10 rows × 3 columns | PASS (code) | 10 outcome rows with Owner / How measured / Current performance columns |

#### Step 4: C — Workforce (19 questions)

| Check | Result | Notes |
|-------|--------|-------|
| 19 questions render | PASS (code) | 19 questions across 6 sub-groups (C1–C6 + catch-all) |
| Sub-group headings visible | **FAIL** | Same as BUG-03 — no sub-group headings |
| Mix of required/optional correct | PASS (code) | C1.1–C4.3 required; C5.x, C6.x, C.catch optional — matches layout spec |

#### Step 5: D — Strategy (13 questions)

| Check | Result | Notes |
|-------|--------|-------|
| 13 questions render | PASS (code) | 13 questions defined |
| SingleSelect (D3.1) OD Stage | PASS (code) | 4 maturity options via `DxRadioGroup` |
| Sub-group headings visible | **FAIL** | No sub-group headings (BUG-03) |

#### Step 6: E — Process & Reality Tests

| Check | Result | Notes |
|-------|--------|-------|
| Process map questions (E1.1–E1.4) | **NOTE** | Layout spec describes compound "FreeTextLong + ResponseMode" questions. Code implements them as separate questions (E1.1 FreeTextLong + E1.1u ResponseMode). Total question count differs from layout (16 in code vs 12 in layout) |
| RatingScaleWithEvidence (E2.1) | PARTIAL | `QRatingScaleWithEvidence.vue` renders rating dropdown + evidence text per item. **BUG-05:** No per-item file upload despite design requiring it. Only rating + evidence text are captured |
| 10 standard rating items | PASS (code) | 10 items defined with 8-point scale |

#### Step 7: F — Operating Cadence (2 questions)

| Check | Result | Notes |
|-------|--------|-------|
| ChecklistUpload (F.grid) 11 schedules | PARTIAL | 11 checklist items render. Same BUG-04 as Step 3 — no actual upload/link collection |
| Catch-all renders | PASS (code) | FreeTextLong |

#### Step 8: G — External System

| Check | Result | Notes |
|-------|--------|-------|
| Compound questions | **NOTE** | Layout says 5 questions (compound FreeTextLong + ResponseMode). Code has 8 separate questions (G.1 + G.1u, G.2 + G.2u, G.3 + G.3u, G.4, G.catch). Functionally equivalent but higher question count |

#### Step 9: H — Finance (31 questions in code)

| Check | Result | Notes |
|-------|--------|-------|
| 28 questions per layout | **31 in code** | Layout spec says 28; instrument has 31 (some compound questions split into separate FreeTextLong + ResponseMode pairs, plus H6.3u added) |
| Two ChecklistUpload blocks | PASS (code) | H5.uploads (7 items) and H6.uploads (5 items) — same BUG-04 applies |

#### Step 10: I — Operations (~47 questions in code)

| Check | Result | Notes |
|-------|--------|-------|
| ~40 questions per layout | **~47 in code** | Additional upload ResponseMode questions split out from compound definitions |
| TableGrid (I5.2) 10 rows | PASS (code) | 10 non-productive time categories with 1 editable column |
| SingleSelect (I6B.1) Yes/No | PASS (code) | 2 options (yes/no) |
| Conditional I6B.2–7 | **FAIL** | **BUG-06:** I6B.2–7 questions always display regardless of I6B.1 answer. `hint: 'Only if you answered Yes above'` is shown but no `showWhen` condition is defined in the instrument. Questions should be conditionally hidden |
| ChecklistUpload (I6B.uploads) | PASS (code) | 5 items — same BUG-04 applies |

#### Step 11: J — Leadership (6 questions)

| Check | Result | Notes |
|-------|--------|-------|
| 6 FreeTextLong questions | PASS (code) | All render as DxTextArea |
| Catch-all at bottom | PASS (code) | J.catch present |
| No sub-group headings needed | PASS | Single group — no sub-groups required |

#### Step 12: K — Evidence (3 questions)

| Check | Result | Notes |
|-------|--------|-------|
| ResponseMode (K.1) | PARTIAL | Same BUG-02 — upload mode not wired |
| TableGrid (K.2) 5 rows × 5 columns | PASS (code) | 5 document rows with name/owner/location/updated/proves columns |
| FileUpload (K.3) | PARTIAL | **BUG-07:** `QFileUpload.vue` emits only `file.name` strings, not `File` objects. With `upload-mode="useForm"`, files are not sent to the backend unless the form is submitted. The wizard uses JSON API calls, not form submission |

#### Step 13: Review & Submit

| Check | Result | Notes |
|-------|--------|-------|
| All sections listed with completion badges | **NOTE** | No dedicated review component found. `isLastSection` triggers "Submit Audit" button instead of "Save and continue". The last section is `section_k`, not a separate review step. **Missing: no Review & Submit step** — the instrument definition has no `review` section |
| Each section expandable with Q&A pairs | **FAIL** | **BUG-08:** No review/summary section implemented. The last wizard step is Section K (Evidence), not a review page |
| "Edit" links navigate back to correct section | N/A | No review section exists |
| "Submit Audit" button visible | PASS (code) | Shows on `isLastSection` (Section K) |
| Submission confirmation appears after submit | PASS (code) | `submitted = true` triggers `.wizard-submitted-card` with success message |

#### Cross-cutting: Navigation & Auto-save

| Check | Result | Notes |
|-------|--------|-------|
| Sidebar shows all sections | PASS (code) | `v-for="section in sections"` with `overflow-y: auto` — handles 13 sections |
| Sidebar click navigates | PASS (code) | `@navigate` emits section key, `navigateToSection()` updates index |
| Completed sections highlighted | PASS (code) | `completedSectionKeys` passed to sidebar, CSS `.completed` class applied |
| Back button works | PASS (code) | `goBack()` decrements `currentSectionIndex` |
| Next button works | PASS (code) | `handleNext()` → `saveCurrentSection()` → `goNext()` |
| Auto-save on answer change | PASS (code) | 2-second debounce via `scheduleAutoSave()` |
| Progress bar shows % | PARTIAL | **BUG-09:** `WizardContextBanner` receives `progress` prop but does NOT render it in the template. Progress is shown in sidebar footer only |
| Drafts loaded on resume | PASS (code) | `loadDrafts()` restores answers and completed sections from backend response |

---

## Issues Register

| ID | Severity | Component | Description |
|----|----------|-----------|-------------|
| BUG-01 | **High** | Consent gate | "Save and continue" button is never disabled on consent step. Clicking with unchecked consent silently does nothing — no visual feedback to the user |
| BUG-02 | **High** | QResponseMode | File upload in "Upload" mode is not bound to model. `DxFileUploader` has no `value` or `@value-changed` binding — selected files are never emitted or stored |
| BUG-03 | **Medium** | All long sections | No sub-group headings (A1/A2/A3/A4, C1–C6, D1–D3, etc.). Dense sections display as flat question lists without visual grouping, making them hard to navigate |
| BUG-04 | **High** | QChecklistUpload | No file upload input or link text field when mode is "upload" or "link". Only the mode radio value is stored — actual files/URLs are not collected |
| BUG-05 | **Medium** | QRatingScaleWithEvidence | No per-item file upload/ResponseMode. Design requires upload capability per rating item; component only has rating dropdown + evidence text |
| BUG-06 | **Medium** | Section I (I6B) | Free-issue questions (I6B.2–7) always display regardless of I6B.1 answer. `showWhen` condition is missing from the instrument definition; only a hint text suggests conditionality |
| BUG-07 | **Medium** | QFileUpload | Emits filenames only (strings), not `File` objects. With JSON API submission (not form-based), actual file data is never sent to the backend |
| BUG-08 | **High** | Review & Submit | No Review & Submit step exists. The instrument definition has no `review` section. The "Submit Audit" button appears on the last data section (K: Evidence) instead of a dedicated review page |
| BUG-09 | **Low** | WizardContextBanner | Progress props passed but not rendered in the banner template. Progress is only visible in the sidebar footer |
| NOTE-01 | **Low** | Catch-all questions | Layout spec says "FreeTextLong + FileUpload" for catch-all questions. Code implements FreeTextLong only — no companion FileUpload. Affects ~12 catch-all questions across all sections |
| NOTE-02 | **Info** | Question counts | Code has more questions than layout spec in several sections (E, G, H, I) because compound "FreeTextLong + ResponseMode" questions in the spec are split into separate question entries in the instrument |
| NOTE-03 | **Low** | Type definitions | `SectionDef` is duplicated in `WizardLayout.vue` and `WizardSidebar.vue` instead of importing from `@/types/wizard` |

---

## Summary

| Step | Tested | Pass | Issues Found |
|------|--------|------|-------------|
| 0 — Consent | Yes (code) | Partial | BUG-01 (consent gate button not disabled) |
| 1 — Company Context | Yes (code) | Partial | BUG-02 (ResponseMode upload), NOTE-01 (catch-all) |
| 2 — A. Org Foundation | Yes (code) | Partial | BUG-02, BUG-03 (no sub-groups), NOTE-01 |
| 3 — B. Org Structure | Yes (code) | Partial | BUG-04 (ChecklistUpload), BUG-03 |
| 4 — C. Workforce | Yes (code) | Partial | BUG-03 (no sub-groups) |
| 5 — D. Strategy | Yes (code) | Partial | BUG-03 (no sub-groups) |
| 6 — E. Process & Reality | Yes (code) | Partial | BUG-02, BUG-05 (RatingScale upload), NOTE-02 |
| 7 — F. Operating Cadence | Yes (code) | Partial | BUG-04 (ChecklistUpload) |
| 8 — G. External System | Yes (code) | Pass | NOTE-02 (question count difference) |
| 9 — H. Finance | Yes (code) | Partial | BUG-04 (ChecklistUpload ×2), NOTE-02 |
| 10 — I. Operations | Yes (code) | Partial | BUG-06 (conditional logic), BUG-04, NOTE-02 |
| 11 — J. Leadership | Yes (code) | Pass | — |
| 12 — K. Evidence | Yes (code) | Partial | BUG-02, BUG-07 (FileUpload) |
| 13 — Review & Submit | Yes (code) | **Fail** | BUG-08 (no review step exists) |

### Next Steps

1. **Manual browser session needed** — This was code-only review. A live browser walkthrough should confirm rendering, styling, and interaction
2. **Fix BUG-01** — Disable "Save and continue" on consent step until checkbox is checked
3. **Fix BUG-02** — Wire `DxFileUploader` in `QResponseMode.vue` to emit files to the model
4. **Fix BUG-04** — Add file upload and link input fields to `QChecklistUpload.vue`
5. **Fix BUG-08** — Add a Review & Submit section to the instrument definition and implement a review component
6. **Fix BUG-06** — Add `showWhen` conditions to I6B.2–7 in the instrument definition
7. **Address BUG-03** — Add sub-group heading metadata to instrument sections and render visual separators
