# Participant Interactions — Feature Area

> **Status:** Planning
> **Created:** 9 March 2026
> **Owner:** limitless-portal-design (Design Tier)
> **Affects:** limitless-portal (Code Tier) — backend handlers, frontend views, database schema

---

## Purpose

This area covers **all channels through which participants interact with the engagement team** during a Limitless Modus diagnostic. The "Submit a Question" button inside wizard flows is the entry point, but the scope is broader: it encompasses the full lifecycle of participant-initiated interactions and consultant responses across multiple channels.

## Scope

| In scope | Out of scope |
|----------|-------------|
| Participant → Consultant questions and replies | Consultant → Participant outbound communications (handled by CommunicationsPage) |
| Ticket tracking and status lifecycle | AI synthesis / diagnostic findings |
| Admin inbox and response UI | Wizard flow design (handled by `wizard/` area) |
| Email notification on new questions | Email templates for invitations/reminders (handled by `emails/`) |
| Channel evaluation (chat, WhatsApp, email) | Authentication and SSO |

## Files in This Area

| File | Purpose | Status |
|------|---------|--------|
| `README.md` | This file — overview, scope, navigation | Current |
| `strategy.md` | Channel evaluation, phased roadmap, strategic decisions | Complete |
| `phase-1-action-plan.md` | Detailed Phase 1 design: UI wireframes, implementation steps, effort estimates | Draft — ready for review |
| `ticket-system.md` | Detailed data model and API contract for the ticket system | Planned |
| `email-channel.md` | Email-based interaction design (Phase 2) | Planned |
| `chat-channel.md` | Live chat evaluation and design (Phase 3) | Planned |
| `whatsapp-channel.md` | WhatsApp integration evaluation (Phase 3+) | Planned |

## Current State (as of 9 March 2026)

### What exists

| Component | Status | Location |
|-----------|--------|----------|
| `SubmitQuestionModal.vue` | Implemented | `limitless-portal/frontend/src/components/wizard/SubmitQuestionModal.vue` |
| `WizardRightPanel.vue` (with Submit button) | Implemented | `limitless-portal/frontend/src/components/wizard/WizardRightPanel.vue` |
| `SubmitQuestion` handler | Implemented | `limitless-portal/internal/handlers/participants.go` |
| `support_questions` table | Implemented | `limitless-portal/migrations/005_support_questions.up.sql` |
| `CreateSupportQuestion` storage | Implemented | `limitless-portal/internal/storage/postgres.go` |

### What is missing

| Gap | Impact |
|-----|--------|
| No email notification when question is submitted | Consultant never knows a question was asked |
| No admin UI to view questions | Questions are stored but invisible to the team |
| No reply mechanism | Consultant cannot respond through the portal |
| No ticket status lifecycle | No way to track open/resolved questions |
| No attachment support | Spec mentions optional screenshot, not implemented |
| No participant view of their questions | Participant sees "submitted" confirmation but cannot check status later |
| No chat infrastructure | Phase 2 per right-panel spec |

## Related Design Specs

- `wizard/_shared/right-panel.md` — Module 4: Support Actions (Submit a Question + Chat with Consultant)
- `architecture/notifications.md` — AWS SES email infrastructure
- `architecture/portal-spec.md` — Overall portal architecture

---

*Part of [Features](../) — [Limitless Portal Design](../../CLAUDE.md)*
