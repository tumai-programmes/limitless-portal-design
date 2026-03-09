# Submission Reset — Skill Implementation

**Created:** 2026-03-08
**Skill location:** `skills/submission-reset/SKILL.md`
**Status:** Implemented and tested

## Overview

A reusable AI skill that resets a Limitless Portal submission to its initial `not_started` state directly from chat. The skill uses the Supabase MCP `execute_sql` tool to run SQL against the production database, following a strict verify-count-reset-confirm workflow.

## How to Invoke

From any Cursor chat session with skills loaded, say:

- "reset submission `<uuid>`"
- "clear submission `<uuid>`"
- "wipe submission `<uuid>`"

The skill activates on these trigger phrases and the presence of a submission UUID.

## Execution Workflow

The skill follows four steps in sequence. Each step uses the Supabase MCP `execute_sql` tool.

### Step 1 — Verify target

Queries `submissions` joined with `engagements` and `participants` to display:

| Field | Purpose |
|-------|---------|
| `id` | Confirm correct UUID |
| `status` | Gate check — `submitted`/`reviewed` require explicit confirmation |
| `progress_percent` | Show how much work will be lost |
| `instrument_type` | Which audit instrument |
| `client_name` | Which engagement/client |
| `participant_name` | Who filled it in |
| `email` | Secondary identification |

### Step 2 — Count deletions

Counts rows in `section_drafts`, `responses`, and `uploads` for the submission. This tells the user exactly what will be destroyed before proceeding.

### Step 3 — Execute reset (5 SQL statements)

| # | Table | Operation | Note |
|---|-------|-----------|------|
| 1 | `uploads` | `DELETE` | DB records only; S3 objects left as orphans |
| 2 | `responses` | `DELETE` | Field-level responses (consultant flow) |
| 3 | `section_drafts` | `DELETE` | Auto-saved participant answers |
| 4 | `submissions` | `UPDATE` | Reset to `not_started`, zero progress, clear all timestamps |
| 5 | `audit_log` | `INSERT` | Record the reset with method `skill` and reason `pilot reset` |

### Step 4 — Confirm

Re-queries the submission to verify `status = 'not_started'`, `progress_percent = 0`, `consent_given = false`, `last_saved_at = null`.

## Safety Rules

1. **Status gate**: Submissions with status `submitted` or `reviewed` trigger a warning. The skill asks for explicit user confirmation before proceeding.
2. **Pre-flight verification**: The target is always displayed before any destructive operation.
3. **Audit trail**: Every reset inserts an `audit_log` row — even when invoked via skill.
4. **S3 untouched**: File objects in S3 are not deleted. Only database references in the `uploads` table are removed. Orphan cleanup is a separate concern (Phase 2).
5. **Single submission**: The skill operates on exactly one UUID per invocation. No bulk operations.

## Anchor Field

The reset is anchored on `submission_id` (UUID). This is the primary key of the `submissions` table and the foreign key referenced by all related tables:

```
submissions.id  ←  section_drafts.submission_id
                ←  responses.submission_id
                ←  uploads.submission_id
```

One UUID cascades the entire reset cleanly.

## First Execution Log

| Field | Value |
|-------|-------|
| **Date** | 2026-03-08 |
| **Submission** | `a0505200-3a47-482d-b0be-4e864fe1a360` |
| **Participant** | Mark Director (mark.director@credo-group.co.uk) |
| **Client** | CREDO GROUP UK LTD |
| **Instrument** | Company Audit |
| **Pre-reset status** | `draft` / 0% progress |
| **Data deleted** | 0 drafts, 0 responses, 0 uploads (submission was empty) |
| **Post-reset status** | `not_started` / 0% / consent false / last_saved null |
| **Outcome** | Clean reset confirmed |

## Alternative Execution Methods

| Method | When to use | Location |
|--------|-------------|----------|
| **Skill (this)** | Routine resets from chat | `skills/submission-reset/SKILL.md` |
| **Admin API** | Programmatic reset from code or curl | `POST /api/admin/submissions/{id}/reset` |
| **Manual SQL** | Direct database access via DataGrip or Supabase console | `sql-hub/limitless-portal/diagnostics/reset-submission.sql` |

All three methods perform the same operations in the same order. The skill and manual SQL log with `method: "skill"` or `method: "manual_sql"` respectively; the API endpoint logs with `method: "api"` via the Go backend's transactional repository method.

## Dependencies

- **Supabase MCP** — `execute_sql` tool must be available in the Cursor session
- **Database access** — the MCP must be connected to the Limitless Portal Supabase project

## Related Files

| File | Purpose |
|------|---------|
| `skills/submission-reset/SKILL.md` | Skill definition (instructions for the AI) |
| `limitless-portal-design/features/submission-reset/README.md` | Full design note (phases, safety, test scenarios) |
| `sql-hub/limitless-portal/diagnostics/reset-submission.sql` | Standalone SQL script for manual execution |
| `limitless-portal/internal/handlers/admin.go` | Go handler for the API endpoint |
| `limitless-portal/internal/storage/postgres.go` | Repository implementation (transactional reset) |
| `limitless-portal/internal/models/participant.go` | `ResetResult` model |
| `limitless-portal/internal/router/router.go` | Route registration |
