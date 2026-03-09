# Email Notifications — Design Spec

*Created: 2026-03-06*
*Updated: 2026-03-08 — added instrument-specific templates from Greg, gap analysis*
*Provider: AWS SES*
*Status: Implemented (backend), not yet connected to UI*

---

## Overview

The portal sends transactional emails to participants and consultants at key moments in the engagement lifecycle. All emails are sent via AWS SES from `portal@limitlessmodus.com`.

**Email copy and content:** See [`../emails/`](../emails/) for Greg's instrument-specific email templates (Director, Manager, Engineer invitations + universal reminder).

## Email Types

### 1. Invitation

| Field | Value |
|-------|-------|
| **Trigger** | Consultant invites a participant to an engagement |
| **Recipient** | Participant (invited user) |
| **Subject** | "You're invited to {engagement_name}" |
| **Content** | Welcome message, brief description of what to expect, CTA button to portal |
| **API** | `POST /api/notifications/invite` |

**Template variables:**
- `recipient_name` — participant's display name
- `engagement_name` — e.g. "Nano Fibre Diagnostic"
- `portal_url` — login URL
- `sender_name` — consultant's name

### 2. Reminder

| Field | Value |
|-------|-------|
| **Trigger** | Manual (consultant sends) or scheduled (future: cron) |
| **Recipient** | Participant with incomplete audit |
| **Subject** | "Reminder: Complete your {instrument_name}" |
| **Content** | Friendly nudge, mention progress is saved, due date, CTA button |
| **API** | `POST /api/notifications/remind` |

**Template variables:**
- `recipient_name` — participant's display name
- `engagement_name` — engagement name
- `instrument_name` — e.g. "Company Audit"
- `due_date` — human-readable due date
- `portal_url` — direct link to continue

### 3. Submission Complete

| Field | Value |
|-------|-------|
| **Trigger** | Participant submits a completed audit |
| **Recipient** | Consultant (Greg) |
| **Subject** | "Submission received: {instrument_name}" |
| **Content** | Who submitted, which instrument, CTA to review |
| **API** | Triggered automatically from submission handler (future) |

**Template variables:**
- `recipient_name` — consultant's name
- `participant_name` — who submitted
- `engagement_name` — engagement name
- `instrument_name` — e.g. "Company Audit"
- `portal_url` — link to review submission

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/notifications/invite` | Consultant only | Send invitation email |
| POST | `/api/notifications/remind` | Consultant only | Send reminder email |

Both endpoints accept JSON body and return `{ "status": "sent", "to": "...", "message": "..." }`.

## UI Integration (Future)

### Engagement Management Page

- **Invite button** next to each participant row → opens modal with pre-filled name/email → calls `/api/notifications/invite`
- **Remind button** next to incomplete submissions → calls `/api/notifications/remind`

### Submission Flow

- On `POST /api/participant/submissions/{id}/submit` → auto-send submission-complete email to consultant (wire into handler)

### Notification Log (Phase 3+)

- Table showing sent notifications per engagement: timestamp, type, recipient, status
- Requires a `notification_log` table in the database

## Email Design

- **Header background:** `#1a237e` (primary, per `design-tokens/colours.md`)
- **CTA button:** `#1a237e` (primary) or `#1565c0` (secondary)
- **Body text:** `#333333` (`--lm-color-text`)
- **Secondary text:** `#666666` (`--lm-color-text-secondary`)
- **Info box background:** `#f8f9fb` (`--lm-color-bg-page`)
- **Border:** `#e0e0e0` (`--lm-color-border`)
- **Font:** Inter / system stack (per `design-tokens/typography.md`)
- **Layout:** Single-column, 600px max width, responsive
- **Footer:** "Limitless Modus — AI-Native Enterprise Transformation"
- **Templates:** Stored in `skills/email-send/templates/` and compiled into Go binary

> **Note:** Previous spec used `#1a1a2e` (header) and `#4f46e5` (CTA). Updated 2026-03-08 to align with the design token system.

## Configuration

| Variable | Value | Notes |
|----------|-------|-------|
| `AWS_SES_REGION` | `eu-west-2` | London region |
| `AWS_SES_FROM_EMAIL` | `portal@limitlessmodus.com` | Verified in SES |
| `AWS_SES_FROM_NAME` | `Limitless Modus` | Display name |
| `PORTAL_URL` | `https://nano.limitlessmodus.com` | Used in CTA buttons |

## DNS Requirements

Before sending, the domain must be verified in AWS SES with:
- SPF record (TXT, include `amazonses.com`)
- DKIM records (3 CNAME records from SES console)
- DMARC record (TXT, `_dmarc.limitlessmodus.com`)

See: `it-hub/integrations/services/aws-ses.md` for full setup steps.

## Dependencies

| What | Where |
|------|-------|
| `aws-sdk-go-v2/service/sesv2` | Go backend dependency |
| `boto3` | Python skill dependency |
| Cloudflare DNS records | Via cloudflare skill |
| AWS IAM `ses:SendEmail` permission | Shared with S3 IAM user |

## Gap Analysis — Greg's Templates vs Current Implementation

*Added 2026-03-08 after receiving Greg's instrument-specific email drafts.*

| Gap | Current state | What Greg's drafts require | Action |
|-----|--------------|---------------------------|--------|
| **Generic invitation only** | Single "you've been invited" template | 3 instrument-specific templates with role-tailored tone and instructions | Add `instrument_type` field to invite request; select template variant |
| **No submission instructions** | CTA button only | Numbered "How to submit" steps per role (5 steps for Director/Manager, 4 for Engineer) | Embed step lists into HTML templates |
| **No confidentiality block** | Not present | GDPR/data-handling paragraph with `[Supplier]` placeholder | Add to all invitation templates |
| **No "Before you start" guidance** | Not present | Reassurance bullets ("Incomplete is OK", "Don't rewrite documents") | Add content block before submission steps |
| **No estimated time** | Not present | Director: 45–60 min, Manager: 30–45 min, Engineer: 10–12 min | Add to info box |
| **Subject lines lack instrument context** | `"You're invited to {engagement_name}"` | Instrument-specific subjects with Invincibility Blueprint® branding | Update subject line generation |
| **Colour mismatch** | `#1a1a2e` header, `#4f46e5` CTA | Design tokens: `#1a237e` primary | Align HTML templates to design tokens |
| **No sender contact details** | Not present | Greg's phone and email in sign-off | Add `sender_phone`, `sender_email` variables |

### Priority actions

1. Extend invitation API to accept `instrument_type` (01, 02, 03) and route to the correct template
2. Create 3 HTML template variants for instrument-specific invitations
3. Add confidentiality paragraph block and submission instructions to templates
4. Align colours to design tokens (`#1a237e` primary)
5. Add sender contact details to sign-off

## Phasing

| Phase | What | Status |
|-------|------|--------|
| **Now** | Go email service + handler + API endpoints | Done |
| **Now** | Email copy from Greg (4 templates) | Done — see [`../emails/`](../emails/) |
| **1C** | Wire invitation to user management UI (task 1.13) | Pending |
| **1C** | Extend templates with instrument-specific content | Pending |
| **2** | Wire reminder to engagement dashboard; auto-send on submission (task 2.4) | Pending |
| **3+** | Notification log table, scheduled reminders, delivery tracking | Future |

---

*Part of the [Portal Architecture](README.md) — [Limitless Portal Design](../README.md).*
