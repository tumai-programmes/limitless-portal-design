# Participant Communications Management

*Created: 2026-03-08*
*Updated: 2026-03-08 — resolved open questions, decisions locked*
*Status: Design spec — approved, pending implementation*

---

## Problem Statement

Consultants (Greg) manage diagnostic engagements where multiple participants — directors, managers, and engineers — are invited to complete audit instruments via the portal. Today, participant records can be created and the backend can send emails, but there is **no way for the consultant to**:

- See at a glance who has been invited, who has started, and who has submitted
- Send or resend invitations from the portal UI
- Track the history of email interactions per participant
- Send targeted reminders to overdue participants
- Perform bulk invitation actions for new engagements

The portal needs a **Participant Tracker** — a dedicated consultant-facing view that closes the gap between the participant CRUD (which exists) and the email service (which exists) by wiring them together through a communication log and a purpose-built UI.

---

## Current State

### What exists

| Component | Status | Location |
|-----------|--------|----------|
| Participants table (DB) | Implemented | `migrations/003_wizard_data_model.up.sql` |
| Participant CRUD endpoints | Implemented | `GET/POST/PUT/DELETE /api/engagements/{id}/participants` |
| Participant grid in UI | Implemented | `EngagementSettingsPage.vue` — add, edit, delete, bulk import |
| AWS SES email service | Implemented | `internal/services/email.go` |
| Invitation API | Implemented | `POST /api/notifications/invite` |
| Reminder API | Implemented | `POST /api/notifications/remind` |
| Instrument-specific HTML templates | Implemented | `skills/email-send/templates/invitation-{director,manager,engineer}.html` |
| Participant wizard & dashboard | Implemented | Participant-facing submission flow |
| Submissions table (DB) | Implemented | Tracks `status`, `started_at`, `submitted_at`, `progress_percent` |

### What is missing

| Gap | Impact |
|-----|--------|
| **No notification_log table** | Cannot track when emails were sent, to whom, or whether they succeeded |
| **No invite/remind buttons in UI** | Consultant cannot trigger emails from the portal |
| **No status derivation** | No way to see a participant's journey: not invited → invited → started → submitted |
| **No communication history** | Cannot see when each email was sent or how many reminders were sent |
| **Notification handler not DB-aware** | Current handler accepts raw email/name — does not reference participant records |
| **No bulk invitation flow** | Cannot invite all uninvited participants in one action |
| **No instrument-aware routing** | API does not select the correct template based on participant's assigned instrument |

---

## Proposed Solution

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│ Frontend (Vue 3 + DevExtreme)                                   │
│                                                                 │
│  EngagementSettingsPage (existing)                              │
│    ├── Settings tab (existing)                                  │
│    ├── Participants tab (existing grid — enhanced)              │
│    └── Communications tab (NEW — Participant Tracker)           │
│           ├── Summary bar (stats: invited / started / submitted)│
│           ├── Participant grid with status columns              │
│           │    ├── Invite / Remind / Resend buttons per row     │
│           │    └── Expand row → communication timeline          │
│           └── Bulk actions toolbar                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ Backend API (Go)                                                │
│                                                                 │
│  Enhanced notification handlers                                 │
│    ├── POST /api/engagements/{id}/invite/{participantId}        │
│    ├── POST /api/engagements/{id}/remind/{participantId}        │
│    ├── POST /api/engagements/{id}/invite-all                    │
│    └── GET  /api/engagements/{id}/communications                │
│                                                                 │
│  notification_log table ← logs every email sent                 │
│  Status derived from: notification_log + submissions            │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ AWS SES                                                         │
│  Templates: invitation-director, invitation-manager,            │
│             invitation-engineer, reminder                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Changes

### New table: `notification_log`

```sql
CREATE TABLE notification_log (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    engagement_id   UUID NOT NULL REFERENCES engagements(id),
    participant_id  UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
    notification_type TEXT NOT NULL,  -- 'invitation' | 'reminder' | 'submission_complete'
    instrument_type TEXT,             -- 'company_audit' | 'manager_audit' | 'engineer_mini_audit'
    channel         TEXT NOT NULL DEFAULT 'email',
    recipient_email TEXT NOT NULL,
    subject         TEXT,
    status          TEXT NOT NULL DEFAULT 'sent',  -- 'sent' | 'failed' | 'bounced' | 'delivered'
    error_message   TEXT,
    sent_by         UUID REFERENCES users(id),     -- consultant who triggered it
    metadata        JSONB DEFAULT '{}',            -- template used, SES message ID, etc.
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notification_log_engagement ON notification_log(engagement_id);
CREATE INDEX idx_notification_log_participant ON notification_log(participant_id);
CREATE INDEX idx_notification_log_type ON notification_log(notification_type);
```

### New columns on existing tables

**`engagements` table** — add default due date:

```sql
ALTER TABLE engagements ADD COLUMN due_date DATE;
```

**`participants` table** — add per-participant due date override:

```sql
ALTER TABLE participants ADD COLUMN due_date_override DATE;
```

When sending emails, the effective due date is: `COALESCE(p.due_date_override, e.due_date)`. If neither is set, the email omits the deadline line.

Invitation status is **derived** from `notification_log` and `submissions` at query time — no status column needed, avoiding stale data.

---

## Status Derivation Logic

A participant's communication status is computed by joining three sources:

```
participants ← notification_log (emails sent)
participants ← submissions     (audit progress)
```

### Status states

| Status | Condition | UI indicator |
|--------|-----------|-------------|
| **Not invited** | No rows in `notification_log` for this participant | Grey dot |
| **Invited** | At least one `invitation` row in `notification_log`, no submission started | Blue dot + "Invited {date}" |
| **Reminded** | At least one `reminder` row after the invitation | Blue dot + "Reminded ×{count}" |
| **Started** | Submission exists with `status = 'in_progress'` and `progress_percent > 0` | Amber dot + "{progress}% complete" |
| **Submitted** | Submission exists with `status = 'submitted'` | Green dot + "Submitted {date}" |
| **Reviewed** | Submission exists with `status = 'reviewed'` | Green check + "Reviewed {date}" |

### Derivation query (conceptual)

```sql
SELECT
    p.id,
    p.name,
    p.email,
    p.role_title,
    p.department,
    p.instruments_assigned,
    COALESCE(p.due_date_override, e.due_date) AS effective_due_date,

    -- Latest invitation
    inv.invited_at,
    inv.invited_by_name,

    -- Reminder count
    rem.reminder_count,
    rem.last_reminded_at,

    -- Submission status
    s.status AS submission_status,
    s.progress_percent,
    s.started_at,
    s.submitted_at,

    -- Derived communication status
    CASE
        WHEN s.status = 'reviewed'    THEN 'reviewed'
        WHEN s.status = 'submitted'   THEN 'submitted'
        WHEN s.status = 'in_progress' THEN 'started'
        WHEN rem.reminder_count > 0   THEN 'reminded'
        WHEN inv.invited_at IS NOT NULL THEN 'invited'
        ELSE 'not_invited'
    END AS communication_status

FROM participants p
JOIN engagements e ON e.id = p.engagement_id

LEFT JOIN LATERAL (
    SELECT
        nl.created_at AS invited_at,
        u.name AS invited_by_name
    FROM notification_log nl
    LEFT JOIN users u ON u.id = nl.sent_by
    WHERE nl.participant_id = p.id
      AND nl.notification_type = 'invitation'
      AND nl.status = 'sent'
    ORDER BY nl.created_at DESC
    LIMIT 1
) inv ON TRUE

LEFT JOIN LATERAL (
    SELECT
        COUNT(*) AS reminder_count,
        MAX(nl.created_at) AS last_reminded_at
    FROM notification_log nl
    WHERE nl.participant_id = p.id
      AND nl.notification_type = 'reminder'
      AND nl.status = 'sent'
) rem ON TRUE

LEFT JOIN LATERAL (
    SELECT s.*
    FROM submissions s
    WHERE s.participant_id = p.id
    ORDER BY s.updated_at DESC
    LIMIT 1
) s ON TRUE

WHERE p.engagement_id = $1
ORDER BY p.name;
```

This query returns everything the frontend needs in a single round trip.

---

## API Design

### New endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET` | `/api/engagements/{id}/communications` | Consultant | List all participants with communication status (derived query above) |
| `POST` | `/api/engagements/{id}/invite/{participantId}` | Consultant | Send invitation to a specific participant |
| `POST` | `/api/engagements/{id}/remind/{participantId}` | Consultant | Send reminder to a specific participant |
| `POST` | `/api/engagements/{id}/invite-all` | Consultant | Bulk invite all uninvited participants |
| `GET` | `/api/engagements/{id}/communications/{participantId}/history` | Consultant | Get communication timeline for one participant |

### Existing endpoints to deprecate (after migration)

| Old endpoint | Replacement | Notes |
|-------------|-------------|-------|
| `POST /api/notifications/invite` | `POST /api/engagements/{id}/invite/{participantId}` | Old endpoint accepts raw email; new one uses participant record |
| `POST /api/notifications/remind` | `POST /api/engagements/{id}/remind/{participantId}` | Same — new one logs to notification_log |

The old endpoints can remain for backward compatibility but should not be used by the new UI.

### POST `/api/engagements/{id}/invite/{participantId}`

**Request:** No body required — all data is derived from the participant record.

Optional overrides (per-participant due date takes precedence over engagement-level default):

```json
{
    "due_date": "2026-04-15",
    "custom_message": "Looking forward to your input."
}
```

**Backend flow:**

1. Load participant from DB (name, email, instruments_assigned)
2. Load engagement from DB (client_name, client_legal_name)
3. Determine instrument type from `instruments_assigned[0]`
4. Select template: `invitation-director`, `invitation-manager`, or `invitation-engineer`
5. Populate template variables (recipient_name, due_date, portal_url, supplier_name, sender_name, sender_email)
6. Send via SES
7. Log to `notification_log` (type=invitation, status=sent/failed, instrument_type, sent_by)
8. Return response

**Response (success):**

```json
{
    "status": "sent",
    "participant_id": "...",
    "notification_type": "invitation",
    "instrument_type": "company_audit",
    "sent_at": "2026-03-08T17:30:00Z"
}
```

**Response (failure):**

```json
{
    "status": "failed",
    "error": "SES rejected: email address not verified in sandbox"
}
```

### POST `/api/engagements/{id}/invite-all`

**Request:**

```json
{
    "due_date": "2026-04-15",
    "filter": "not_invited"
}
```

`filter` options: `not_invited` (default), `not_started`, `all`.

**Backend flow:**

1. Load all participants for engagement
2. Filter by status (default: those with no invitation log entry)
3. For each participant: determine instrument → select template → send → log
4. Return summary

**Response:**

```json
{
    "total": 12,
    "sent": 11,
    "failed": 1,
    "failures": [
        { "participant_id": "...", "email": "...", "error": "..." }
    ]
}
```

### GET `/api/engagements/{id}/communications`

Returns the derived status view. This is the primary data source for the Participant Tracker grid.

**Response:**

```json
{
    "engagement_id": "...",
    "summary": {
        "total": 15,
        "not_invited": 3,
        "invited": 4,
        "reminded": 2,
        "started": 3,
        "submitted": 2,
        "reviewed": 1
    },
    "participants": [
        {
            "id": "...",
            "name": "John Smith",
            "email": "john@example.com",
            "role_title": "Managing Director",
            "department": "",
            "instruments_assigned": ["company_audit"],
            "communication_status": "invited",
            "invited_at": "2026-03-05T10:00:00Z",
            "invited_by": "Greg Kurnikov",
            "reminder_count": 0,
            "last_reminded_at": null,
            "submission_status": null,
            "progress_percent": 0,
            "started_at": null,
            "submitted_at": null
        }
    ]
}
```

### GET `/api/engagements/{id}/communications/{participantId}/history`

Returns the full communication timeline for one participant.

**Response:**

```json
{
    "participant": {
        "id": "...",
        "name": "John Smith",
        "email": "john@example.com"
    },
    "events": [
        {
            "type": "invitation",
            "channel": "email",
            "status": "sent",
            "sent_by": "Greg Kurnikov",
            "timestamp": "2026-03-05T10:00:00Z",
            "subject": "Invincibility Blueprint® — Company Audit for Nano Fibre"
        },
        {
            "type": "reminder",
            "channel": "email",
            "status": "sent",
            "sent_by": "Greg Kurnikov",
            "timestamp": "2026-03-12T09:00:00Z",
            "subject": "Reminder: Your Company Audit is due soon"
        },
        {
            "type": "submission_started",
            "channel": "system",
            "timestamp": "2026-03-13T14:22:00Z"
        },
        {
            "type": "submission_complete",
            "channel": "system",
            "timestamp": "2026-03-15T16:45:00Z"
        }
    ]
}
```

Note: `submission_started` and `submission_complete` events are derived from the `submissions` table, not from `notification_log`. The API merges both sources into a single timeline.

---

## Frontend Design

### Navigation

The Participant Tracker is a **standalone page** in the sidebar navigation, accessible within the engagement context:

```
📊 Dashboard
📋 Company Audit  (01)
📋 Manager Audit  (02)
📋 Engineer Audit (03)
📧 Communications  ← NEW
⚙️ Settings
```

**Decision:** Standalone nav item in the sidebar. Communications is a first-class feature, not buried in settings.

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Communications                                              │
│                                                             │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│ │    15    │ │     4   │ │     6   │ │     5   │           │
│ │  Total   │ │ Invited │ │ Started │ │Submitted│           │
│ └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
│                                                             │
│ ┌─────────────────────────────────┐                         │
│ │ 🔍 Search...    [Invite All ▼]  │ ← Bulk action dropdown │
│ └─────────────────────────────────┘                         │
│                                                             │
│ ┌────────────────────────────────────────────────────────┐  │
│ │ Name      │ Email    │ Instrument │ Status │ Invited  │  │
│ │           │          │            │        │          │  │
│ ├───────────┼──────────┼────────────┼────────┼──────────┤  │
│ │ J. Smith  │ j@...    │ 01 Company │ ● Inv  │ 5 Mar    │  │
│ │  ▶ View history                                       │  │
│ ├───────────┼──────────┼────────────┼────────┼──────────┤  │
│ │ A. Brown  │ a@...    │ 02 Manager │ ● Sta  │ 5 Mar    │  │
│ ├───────────┼──────────┼────────────┼────────┼──────────┤  │
│ │ T. Lee    │ t@...    │ 03 Eng     │ ○ --   │ --       │  │
│ │           │          │            │        │ [Invite] │  │
│ └────────────────────────────────────────────────────────┘  │
│                                                             │
│ Expanded row (J. Smith):                                    │
│ ┌────────────────────────────────────────────────────────┐  │
│ │ Timeline:                                              │  │
│ │ ● 5 Mar 10:00  Invitation sent (by Greg)              │  │
│ │ ● 12 Mar 09:00 Reminder sent (by Greg)         [Remind]│  │
│ │ ● 13 Mar 14:22 Started audit (22% complete)           │  │
│ └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Summary cards

Four status cards at the top, styled as DevExtreme tiles with counts:

| Card | Count source | Colour |
|------|-------------|--------|
| **Total** | All participants | Neutral (grey) |
| **Invited** | `communication_status IN ('invited', 'reminded')` | Blue (`#1565c0`) |
| **Started** | `communication_status = 'started'` | Amber (`#f59e0b`) |
| **Submitted** | `communication_status IN ('submitted', 'reviewed')` | Green (`#16a34a`) |

### Grid columns

| Column | Source | Width | Notes |
|--------|--------|-------|-------|
| **Name** | `participants.name` | auto | Sortable |
| **Email** | `participants.email` | auto | Sortable |
| **Instrument** | `instruments_assigned` | 140px | Badge: "01 Company", "02 Manager", "03 Engineer" |
| **Status** | Derived `communication_status` | 130px | Coloured dot + label |
| **Invited** | `notification_log` | 100px | Date of first invitation, or "--" |
| **Reminders** | `notification_log` | 90px | Count (e.g. "×2") or "--" |
| **Progress** | `submissions.progress_percent` | 100px | Progress bar or "--" |
| **Submitted** | `submissions.submitted_at` | 100px | Date or "--" |
| **Actions** | — | 120px | Context-sensitive buttons |

### Action buttons (per row)

| Participant status | Available actions |
|--------------------|-------------------|
| **Not invited** | [Invite] |
| **Invited** | [Remind] [Resend] |
| **Reminded** | [Remind] |
| **Started** | [Remind] |
| **Submitted** | (no email actions) |
| **Reviewed** | (no email actions) |

"Resend" sends the same invitation template again. "Remind" sends the reminder template.

### Expanded row — communication timeline

Clicking on a row expands it to show the full chronological history (from `GET .../history`). Each event is a line with:

- Timestamp (relative: "3 days ago" or absolute: "5 Mar 10:00")
- Event type icon (✉ email sent, 🔔 reminder, ▶ started, ✓ submitted)
- Description ("Invitation sent by Greg", "Reminder sent by Greg", "Started audit — 22% complete")

### Bulk actions

The "Invite All" dropdown button offers:

| Action | Description |
|--------|-------------|
| **Invite all uninvited** | Sends invitation to every participant with no `notification_log` entry |
| **Remind all overdue** | Sends reminder to everyone invited but not yet submitted |
| **Remind not started** | Sends reminder to everyone invited but who hasn't opened the audit |

Each bulk action shows a confirmation dialog: "Send invitations to 5 participants?" with a summary list.

### DevExtreme components used

| Component | Usage |
|-----------|-------|
| `DxDataGrid` | Main participant tracker grid |
| `DxColumn` | Grid columns with custom cell templates |
| `DxMasterDetail` | Expandable row for communication timeline |
| `DxButton` | Action buttons, bulk actions |
| `DxDropDownButton` | "Invite All" dropdown menu |
| `DxLoadPanel` | Loading state during API calls |
| `DxToast` | Success/failure notifications after send |
| `DxPopup` | Confirmation dialogs for bulk actions |

---

## Instrument-Type Routing

The backend determines which email template to use based on the participant's `instruments_assigned` field:

| `instruments_assigned` value | Template | Subject |
|-----------------------------|----------|---------|
| `["company_audit"]` | `invitation-director.html` | "Invincibility Blueprint® — Company Audit for {client}" |
| `["manager_audit"]` | `invitation-manager.html` | "Invincibility Blueprint® — Your Manager Audit for {client}" |
| `["engineer_mini_audit"]` | `invitation-engineer.html` | "Invincibility Blueprint® — Quick Survey for {client}" |
| (any) | `reminder.html` | "Reminder: Your {instrument} audit is due soon" |

If a participant has multiple instruments assigned, send one invitation per instrument (rare edge case — typically each participant has exactly one).

---

## Rate Limiting and Safeguards

| Rule | Description |
|------|-------------|
| **No duplicate invitation within 24h** | If an invitation was sent to this participant in the last 24 hours, block with warning |
| **Max 3 reminders per participant** | After 3 reminders, show warning "Maximum reminders reached — contact participant directly" |
| **Bulk send throttle** | SES rate limit is 14/sec; batch sends with 100ms delay between emails |
| **Confirmation required for bulk** | All bulk actions show a confirmation dialog before sending |
| **Sent-by attribution** | Every log entry records which consultant triggered the send |

---

## Implementation Phases

### Phase 1 — Foundation (next sprint)

| # | Task | Type | Effort |
|---|------|------|--------|
| 1.1 | Create `notification_log` migration + `due_date` columns on engagements/participants | Backend | Small |
| 1.2 | Add `notification_log` repository methods (Create, List, GetByParticipant) | Backend | Small |
| 1.3 | Refactor notification handler to accept `participantId`, load from DB, log to `notification_log` | Backend | Medium |
| 1.4 | Add `POST /api/engagements/{id}/invite/{participantId}` endpoint | Backend | Medium |
| 1.5 | Add `POST /api/engagements/{id}/remind/{participantId}` endpoint | Backend | Small |
| 1.6 | Add `GET /api/engagements/{id}/communications` endpoint (derived status query) | Backend | Medium |
| 1.7 | Create `CommunicationsPage.vue` as standalone page + add sidebar nav item + route | Frontend | Medium |
| 1.8 | Build participant tracker DataGrid with status columns | Frontend | Medium |
| 1.9 | Wire Invite/Remind buttons per row | Frontend | Small |
| 1.10 | Add summary cards (total, invited, started, submitted) | Frontend | Small |

### Phase 2 — Enrichment

| # | Task | Type | Effort |
|---|------|------|--------|
| 2.1 | Add `POST /api/engagements/{id}/invite-all` bulk endpoint | Backend | Medium |
| 2.2 | Build bulk action dropdown + confirmation dialog | Frontend | Medium |
| 2.3 | Add `GET .../communications/{participantId}/history` endpoint | Backend | Small |
| 2.4 | Build expandable row with communication timeline (DxMasterDetail) | Frontend | Medium |
| 2.5 | Add rate limiting (24h dedup, max 3 reminders) | Backend | Small |
| 2.6 | Auto-send submission-complete email to consultant on submit | Backend | Small |

### Phase 3 — Advanced

| # | Task | Type | Effort |
|---|------|------|--------|
| 3.1 | SES delivery tracking (webhook for bounce/complaint) | Backend | Large |
| 3.2 | Scheduled reminders (cron: auto-remind 3 days before deadline) | Backend | Medium |
| 3.3 | Email preview before send | Frontend | Medium |
| 3.4 | CSV export of communication status | Frontend | Small |

---

## Decisions (Resolved 2026-03-08)

| # | Question | Decision | Notes |
|---|----------|----------|-------|
| 1 | **Tab or standalone page?** | **Standalone nav item** | Communications is a first-class feature in the sidebar, not a tab buried in Settings |
| 2 | **Due date storage** | **Both: per-engagement default + per-participant override** | Engagement has a default `due_date`; participants can have an individual `due_date_override` that takes precedence |
| 3 | **Sender email** | **`portal@limitlessmodus.com`** | Already configured in SES. System sends on behalf of the portal; Greg's name and contact details appear in the email body sign-off |
| 4 | **notification_log vs audit_log** | **Separate `notification_log` table** | Email-specific fields (subject, SES message ID, recipient, instrument_type) don't fit `audit_log` |
| 5 | **SES sandbox exit** | Pending — verify domain and request production access | Required before real emails can be sent |

---

## Cross-References

| Document | Relevance |
|----------|-----------|
| [architecture/notifications.md](../architecture/notifications.md) | Email types, API design, SES config, gap analysis |
| [architecture/portal-spec.md](../architecture/portal-spec.md) | Data model, participants table, submissions, API endpoints |
| [architecture/status.md](../architecture/status.md) | Phase 1C tasks, what's built vs pending |
| [emails/README.md](../emails/README.md) | Greg's email templates, placeholder mapping, layout structure |
| [wizard/specification.md](../wizard/specification.md) | Instrument definitions, participant roles |
| [design-tokens/](../design-tokens/) | Colours, typography, spacing for UI components |

---

*Part of [Limitless Portal Design](../README.md). See [architecture/notifications.md](../architecture/notifications.md) for the underlying email service spec.*
