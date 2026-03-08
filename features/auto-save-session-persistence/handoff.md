# Auto-Save & Session Persistence — Handoff Note

**Created:** 2026-03-08
**Origin chat:** [File upload pipeline & fixes](48c2dc33-6e77-4826-a4fe-fb4735c82129)
**Status:** Ready for implementation

## Problem Statement

The Limitless Portal wizard needs reliable auto-save and session persistence so that participants can:
- Close the browser mid-wizard and resume exactly where they left off
- Never lose typed answers due to accidental tab closure, network loss, or timeout
- See clear visual feedback about save state (saved, saving, unsaved changes)
- Complete the audit across multiple sessions over days or weeks

The auto-save mechanism is **partially implemented** but has several bugs and gaps that prevent reliable session resumption. The core infrastructure (debounced save, section drafts table, progress tracking) exists but the round-trip is broken.

## What Already Exists

### Frontend — `frontend/src/stores/wizard.ts`

The Pinia wizard store has:

| Feature | Status | Notes |
|---------|--------|-------|
| `answers` reactive state | Working | `Record<string, Record<string, any>>` per section |
| `completedSections` tracking | Working | Array of section key strings |
| `currentSectionIndex` | Working | Tracks active wizard step |
| `scheduleAutoSave()` | Working | 2-second debounce on `setAnswer()` |
| `saveCurrentSection()` | Partially working | Calls two APIs (save section + update progress) |
| `loadDrafts()` | Broken | Key mismatch prevents draft restoration |
| `beforeunload` handler | Not implemented | No save-on-close protection |
| `visibilitychange` handler | Not implemented | No save-on-tab-switch |

### Backend — `internal/handlers/participants.go`

Participant API endpoints:

| Method | Path | Purpose | Status |
|--------|------|---------|--------|
| `GET` | `/api/participant/submissions/{id}` | Load submission + drafts | Incomplete response |
| `PUT` | `/api/participant/submissions/{id}/sections` | Save section draft | Working |
| `PUT` | `/api/participant/submissions/{id}/progress` | Update progress | Format mismatch |
| `POST` | `/api/participant/submissions/{id}/consent` | Record consent | Working |
| `POST` | `/api/participant/submissions/{id}/submit` | Final submission | Working |

### Database — `section_drafts` table (migration 003)

```sql
CREATE TABLE section_drafts (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    submission_id UUID NOT NULL REFERENCES submissions(id),
    section_key   TEXT NOT NULL,
    answers       JSONB NOT NULL DEFAULT '{}',
    is_complete   BOOLEAN NOT NULL DEFAULT false,
    last_saved_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE(submission_id, section_key)
);
```

The `submissions` table also has:
- `status` (draft / in_progress / submitted / completed)
- `progress_percent` (integer)
- `sections_completed` (JSONB array of section keys)
- `consent_given` (boolean)
- `last_saved_at` (timestamp)

## Known Bugs (Must Fix)

### Bug 1: GetSubmission response incomplete

`ParticipantHandler.GetSubmission` returns only `submission_id` and `section_drafts`. It does **not** return `status`, `client_name`, `logo_url`, `consent_given`, or `sections_completed`. The frontend expects all of these for proper resumption.

**Impact:** Participant cannot resume a session — the wizard opens blank.

### Bug 2: Drafts key mismatch

- Backend returns: `section_drafts`
- Frontend expects: `subData.drafts`

`loadDrafts()` looks for `subData.drafts` but the response has `section_drafts`, so drafts are never loaded on resumption.

**Impact:** All previously saved answers are invisible when resuming.

### Bug 3: Answers format mismatch

Backend `SectionDraft.Answers` is stored as JSONB but may be returned as a JSON string. Frontend `loadDrafts()` expects a parsed object. If it receives a string, answers won't populate the form fields.

**Impact:** Even if Bug 2 is fixed, answers may not render correctly.

### Bug 4: Progress payload format

Frontend sends `sections_completed: JSON.stringify(completedSections.value)` (a string). Backend `UpdateSubmissionProgress` expects `sections_completed` as `[]string`. The stringified JSON array may be stored as a single string rather than a proper array.

**Impact:** Completed sections list may be corrupted on save, breaking the progress bar and section completion indicators.

## Missing Features (Should Implement)

### Feature 1: Save on tab close / visibility change

Add `beforeunload` and `visibilitychange` event listeners that trigger an immediate (non-debounced) save of the current section if there are unsaved changes.

### Feature 2: Save status indicator

Show a visual indicator in the wizard UI:
- "All changes saved" (with timestamp) — green/subtle
- "Saving..." — during save
- "Unsaved changes" — when debounce is pending
- "Save failed — retrying" — on error with automatic retry

### Feature 3: Conflict detection

If a participant opens the same submission in two browser tabs, the second save could overwrite the first. Consider:
- Optimistic locking via `last_saved_at` comparison
- Or simply warn "This submission is open in another tab"

### Feature 4: Offline resilience

Queue saves locally (IndexedDB or localStorage) when offline and sync when connectivity returns. This is a nice-to-have, not critical for MVP.

## Key Files

### Frontend

| File | Role |
|------|------|
| `frontend/src/stores/wizard.ts` | Wizard state, auto-save, save/load logic |
| `frontend/src/views/CompanyAuditWizard.vue` | Company Audit wizard (instrument 01) |
| `frontend/src/views/ManagerAuditWizard.vue` | Manager Audit wizard (instrument 02) |
| `frontend/src/views/EngineerMiniAuditWizard.vue` | Engineer Mini-Audit wizard (instrument 03) |
| `frontend/src/views/ParticipantDashboard.vue` | Dashboard with submission cards |

### Backend

| File | Role |
|------|------|
| `internal/handlers/participants.go` | All participant API handlers (save, load, submit) |
| `internal/storage/postgres.go` | `UpsertSectionDraft`, `GetSectionDrafts`, `UpdateSubmissionProgress` |
| `internal/models/participant.go` | `SectionDraft`, `SaveSectionRequest`, `ProgressRequest` |

### Migrations

| File | Role |
|------|------|
| `migrations/001_initial_schema.up.sql` | `submissions` table (base) |
| `migrations/003_wizard_data_model.up.sql` | `section_drafts`, `participants`, submission extensions |

## Architecture Note: Two Submission Systems

The codebase has two separate save paths:

1. **Participant flow** (section_drafts + participant handlers) — used by audit participants
2. **Consultant flow** (responses table + SaveResponse handler) — field-level saves by consultants

These should eventually converge or at least share the same underlying storage, but for this feature work, focus on the **participant flow** only.

## Suggested Implementation Order

1. **Fix Bug 1** — Enrich `GetSubmission` response with all fields the frontend needs
2. **Fix Bug 2** — Align the JSON key names between backend and frontend
3. **Fix Bug 3** — Ensure answers are always returned as parsed objects
4. **Fix Bug 4** — Fix `sections_completed` serialisation
5. **Test round-trip** — Save section → close browser → reopen → verify all answers restored
6. **Feature 1** — Add `beforeunload` / `visibilitychange` save triggers
7. **Feature 2** — Add save status indicator to wizard UI

## Test Scenarios

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Fill Section A, click Next, close browser, reopen | Section A answers restored, wizard opens at Section B |
| 2 | Fill Section A partially, close browser (no Next) | Section A partial answers restored via auto-save |
| 3 | Fill Section A, lose network, type more, network returns | All answers eventually saved |
| 4 | Complete 5 of 11 sections, return next day | Progress bar shows ~45%, wizard opens at Section F |
| 5 | Upload file in Section B, close and return | File upload record persists, shown in upload list |
| 6 | Two browser tabs on same submission | No data loss or corruption |

## Environment

- **Live URL:** https://portal.limitlessmodus.com/
- **Dev URL:** https://dev-portal.limitlessmodus.com/
- **Database:** Supabase (managed PostgreSQL)
- **Auth:** Microsoft Entra ID SSO (dev-mode bypass with `dev-` JWT prefix)
- **Relevant test account:** Participant on engagement `b6d56792-7ef1-45d7-959d-09c9a655d6dc`
