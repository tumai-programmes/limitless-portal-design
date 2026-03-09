# Feature: Submission Reset

Reset a participant's submission (and optionally their communication history) to enable a complete re-run of the engagement workflow.

## Problem

During pilot testing, participants go through the full lifecycle: pre-invite → invite → audit completion → reminders → thank-you. To re-test this end-to-end, we need to revert a participant to a clean slate — not just their submission answers, but also the communication trail that drives the portal's status derivation.

## Two Reset Modes

### Mode 1: Submission-Only Reset (default)

Clears submission data while preserving communication history.

| Table | Action |
|-------|--------|
| `uploads` | DELETE rows for submission |
| `responses` | DELETE rows for submission |
| `section_drafts` | DELETE rows for submission |
| `submissions` | Reset to `not_started`, progress 0, clear all timestamps |
| `notification_log` | **Untouched** |
| `audit_log` | New entry added |

**Use when:** Participant needs to redo the audit form, but you don't need to re-test the email invitation flow.

**Communication status after reset:** Stays at its previous value (e.g. `invited`, `reminded`). The participant can immediately re-open and fill in the audit.

### Mode 2: Full Participant Reset (submission + communications)

Clears both submission data and all communication logs for the participant within the engagement, allowing the entire lifecycle to be re-run from scratch.

| Table | Action |
|-------|--------|
| `uploads` | DELETE rows for submission |
| `responses` | DELETE rows for submission |
| `section_drafts` | DELETE rows for submission |
| `submissions` | Reset to `not_started`, progress 0, clear all timestamps |
| `notification_log` | DELETE rows for participant + engagement |
| `audit_log` | New entry added |

**Use when:** You need to re-test the full communication flow (pre-invite → invite → reminders → completion) from the beginning.

**Communication status after reset:** Returns to `not_invited`. The portal's Communications page will show the participant as if they've never been contacted.

## How Communication Status is Derived

The portal does not store `communication_status` directly. It is computed at query time from `notification_log` and `submissions`:

| Derived Status | Condition |
|----------------|-----------|
| `submitted` | Submission status = `submitted` |
| `started` | Submission status = `started` / `in_progress` |
| `reminded` | At least one `reminder` in notification_log |
| `invited` | At least one `invitation` in notification_log |
| `pre_invited` | At least one `pre_invitation` in notification_log |
| `not_invited` | No matching notification_log rows |

This is why deleting `notification_log` rows is sufficient to reset the communication status — no separate status field needs updating.

## Safety Rules

1. **Never run** on production engagements without explicit confirmation
2. **Always verify** the target — show participant name, email, client, and current status before executing
3. **Always audit** — every reset is logged in `audit_log` with method and reason
4. **Scope to engagement** — communication log deletion is scoped to `participant_id + engagement_id`, never a blanket wipe
5. S3 objects are **not deleted** — only database records are removed

## Tables Affected

| Table | Key Column | Scope |
|-------|-----------|-------|
| `uploads` | `submission_id` | Per submission |
| `responses` | `submission_id` | Per submission |
| `section_drafts` | `submission_id` | Per submission |
| `submissions` | `id` | Per submission |
| `notification_log` | `participant_id` + `engagement_id` | Per participant per engagement |
| `audit_log` | — | Insert only (never deleted) |

## Implementation

| Artefact | Location |
|----------|----------|
| AI Skill | `skills/submission-reset/SKILL.md` |
| Manual SQL | `sql-hub/limitless-portal/diagnostics/reset-submission.sql` |
| Go API endpoint | `POST /api/admin/submissions/{id}/reset` (submission-only; comms reset not yet in API) |

## Future Considerations

- The Go backend `ResetSubmission()` could be extended with a `?include_communications=true` query parameter
- A dedicated "Reset Participant" button in the Communications UI for pilot/test engagements
- Batch reset: reset all participants in an engagement at once for full pilot re-runs
