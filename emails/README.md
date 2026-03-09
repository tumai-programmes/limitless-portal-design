# Email Templates

*Source: Greg Kurnikov, March 2026*
*Status: Draft copy — pending design implementation*

---

## Overview

These templates define the emails sent to participants during the Invincibility Blueprint diagnostic. Each email maps to a specific wizard instrument and participant role. The portal sends them via AWS SES from `portal@limitlessmodus.com`.

## Template Index

| # | Template | Recipient | Instrument | Trigger |
|---|----------|-----------|------------|---------|
| 01 | [Director Invitation](01-director-invitation.md) | Company Director / Owner | 01 — Company Audit | Consultant invites participant |
| 02 | [Manager Invitation](02-manager-invitation.md) | Department / Area Manager | 02 — Managers' Audit | Consultant invites participant |
| 03 | [Engineer Invitation](03-engineer-invitation.md) | Field Engineer (sample) | 03 — Engineer Mini-Audit | Consultant invites participant |
| 04 | [Reminder](04-reminder.md) | Any participant | Any instrument | Manual or scheduled nudge |

## Placeholder Reference

Greg's original placeholders map to the portal's template variables:

| Greg's placeholder | Portal variable | Source |
|--------------------|----------------|--------|
| `[NAME]` | `{{recipient_name}}` | Participant record |
| `[DEADLINE]` | `{{due_date}}` | Engagement settings |
| `[DIRECTOR_FORM_LINK]` | `{{portal_url}}` | Config (`PORTAL_URL`) |
| `[MANAGER_FORM_LINK]` | `{{portal_url}}` | Config (`PORTAL_URL`) |
| `[ENGINEER_FORM_LINK]` | `{{portal_url}}` | Config (`PORTAL_URL`) |
| `[FORM_LINK]` | `{{portal_url}}` | Config (`PORTAL_URL`) |
| `[Supplier legal name]` | `{{supplier_name}}` | Engagement settings (future) |
| `[Supplier]` | `{{supplier_name}}` | Engagement settings (future) |
| `[name/email]` | `{{sender_name}}` / `{{sender_email}}` | Consultant record |
| `[phone]` | `{{sender_phone}}` | Consultant record (future) |
| `[email]` | `{{sender_email}}` | Consultant record |
| *(new)* | `{{company_logo_url}}` | Engagement settings; pilot: hard-coded Nano Fibre URL |

## Design Spec

See [architecture/notifications.md](../architecture/notifications.md) for the technical implementation (API endpoints, SES config, phasing).

Email layout and design tokens are defined in:
- [design-tokens/colours.md](../design-tokens/colours.md)
- [design-tokens/typography.md](../design-tokens/typography.md)
- [design-tokens/spacing.md](../design-tokens/spacing.md)

## Layout Structure

All emails follow a consistent single-column layout at 600px max width:

1. **Header** — Dark navy (`#1a237e`) co-branded: "LIMITLESS MODUS" wordmark + "Invincibility Blueprint" subtitle on the left, vertical separator, client company logo (`{{company_logo_url}}`) on the right
2. **Greeting** — Personalised salutation
3. **Info Box** — Light grey surface with form link, deadline, and estimated time
4. **Instructions** — "Before you start" bullet points + "How to submit" numbered steps
5. **CTA Button** — Primary action button (centred)
6. **Confidentiality** — Data handling notice (smaller text)
7. **Sign-off** — Consultant name, phone, email
8. **Footer** — "Limitless Modus — AI-Native Enterprise Transformation"

---

*Part of [Limitless Portal Design](../README.md). Technical spec: [notifications.md](../architecture/notifications.md).*
