# Participant Interactions — Strategy

> **Status:** Draft
> **Created:** 9 March 2026
> **Decision status:** Open — awaiting review

---

## Context

Participants filling out Limitless Modus audit wizards need a way to ask questions, get clarification, and communicate with the engagement consultant (Greg or team). The portal currently has a "Submit a Question" form that stores questions in the database — but nobody sees them, nobody is notified, and nobody can respond.

This document evaluates four interaction channels, recommends a phased approach, and defines the strategic direction.

## The Four Channels

### Channel 1: Ticket System (Submit a Question → Track → Respond)

**What it is:** A structured question/answer system embedded in the portal. Participants submit questions tagged with their wizard section context. Consultants view, respond, and close tickets through an admin interface. All interactions are tracked and auditable.

**Current state:** Partially implemented — the submission form and database storage exist. Everything else is missing.

**Strengths:**
- Asynchronous — participants and consultants work at different times
- Contextual — questions are auto-tagged with wizard section, submission ID, and participant identity
- Auditable — full history preserved for each engagement
- Low friction — already partially built
- Self-contained — no external dependencies

**Weaknesses:**
- Not real-time — participant must wait for a response
- Requires portal login to check for responses (unless we add email notifications)
- Could feel impersonal compared to chat

**Strategic assessment:** This is the foundation. Every other channel either feeds into or complements the ticket system.

**Recommendation:** **Build first. Complete what exists.**

---

### Channel 2: Live Chat

**What it is:** Real-time messaging between participant and consultant, embedded in the portal wizard.

**Options evaluated:**

| Option | Effort | Cost | Control | UX |
|--------|--------|------|---------|-----|
| **Custom WebSocket chat** | High (2-3 weeks) | None (self-hosted) | Full | Seamless integration |
| **DevExtreme DxChat component** | Medium (1 week) | Included in license | Good | Native look/feel |
| **Third-party widget** (Intercom, Crisp, Tawk.to) | Low (hours) | $0-50/mo | Limited | Foreign UI inside portal |
| **Microsoft Teams integration** | Medium (1 week) | Included in M365 | Medium | Familiar but context-switches |

**Strengths:**
- Immediate answers — reduces frustration and abandonment
- Personal — feels like having a consultant in the room
- Can handle nuanced clarifications faster than async tickets

**Weaknesses:**
- Requires someone to be online to respond (availability problem)
- With <20 active participants at a time, a consultant is unlikely to be online when a question is asked
- Unanswered chat messages are worse UX than a ticket that says "we'll respond shortly"
- Significant infrastructure for low usage

**Strategic assessment:** Premature for current scale. With Greg as the single consultant and engagements running asynchronously across time zones, the chance of real-time alignment is low. A chat that goes unanswered is worse than a ticket system that sets correct expectations.

**Recommendation:** **Defer to Phase 3.** Revisit when concurrent participant count exceeds ~50 or when a dedicated support person is available. If implemented, prefer the DevExtreme DxChat component for consistency.

---

### Channel 3: WhatsApp Integration

**What it is:** Participants can message the engagement team via WhatsApp. Messages are tracked in the portal.

**Options evaluated:**

| Option | Effort | Cost | Requirements |
|--------|--------|------|-------------|
| **WhatsApp Business API** (via Meta Cloud API) | High (2-3 weeks) | ~$0.05/conversation | Meta Business verification, phone number, webhook server |
| **Via Twilio** | High (2 weeks) | ~$0.005/msg + $15/mo | Twilio account, webhook server |
| **Manual WhatsApp** (no integration) | None | None | Greg shares his number, no tracking |

**Strengths:**
- Highest engagement rate — people check WhatsApp constantly
- Familiar — no login required, works on phone
- Push notifications built in
- Greg could respond from his phone anywhere

**Weaknesses:**
- Complex integration — Meta Business verification, template approval, webhook handling
- Blurs personal/business boundaries — Greg's personal WhatsApp vs. business number
- No wizard section context — participant messages lose the structured tagging
- Compliance and data retention concerns (GDPR — messages on Meta's infrastructure)
- Two-way sync complexity — keeping portal ticket state in sync with WhatsApp threads

**Strategic assessment:** High engagement value but wrong timing. The integration complexity is disproportionate to the current participant count. However, the *manual* approach (Greg shares a WhatsApp number, participants message informally) is already available and requires no development.

**Recommendation:** **Acknowledge as a future channel. Do not build integration now.** If Greg is already using WhatsApp manually with participants, that's fine — but don't invest in platform integration until the ticket system is mature and scale demands it.

---

### Channel 4: Email-Based Interaction (tracked by ticket)

**What it is:** When a participant submits a question, the consultant receives an email notification. The consultant can respond via the portal (which sends an email to the participant) or via email reply (which is parsed back into the ticket).

**Current state:** AWS SES is set up for outbound emails. Email tracking (open/click) is implemented. Inbound email handling does not exist.

**Options evaluated:**

| Option | Effort | Complexity |
|--------|--------|-----------|
| **Outbound-only notifications** — consultant gets email alert, responds in portal | Low (hours) | Low — just add SES send on question submit |
| **Outbound + email CTA** — email contains a "Respond in Portal" button linking to the ticket | Low (hours) | Low — template + deep link |
| **Full inbound parsing** — reply-to address routes to the ticket, parsed via SES/SNS | High (1-2 weeks) | High — inbound email handling, DKIM, spam filtering |

**Strengths:**
- Universal — everyone has email, no app install
- Non-intrusive — fits existing workflows
- Pairs naturally with the ticket system
- Leverages existing AWS SES infrastructure

**Weaknesses:**
- Slow feedback loop (email notification → consultant opens portal → responds → participant gets email)
- Inbound parsing is fragile (email clients add signatures, threads get messy)
- Not real-time

**Strategic assessment:** Email notification is the critical missing piece that makes the ticket system actually work. Without it, questions go into a black hole. Full inbound parsing is complex and fragile — not worth it.

**Recommendation:** **Implement outbound notifications immediately as part of Phase 1.** Add a "Respond in Portal" deep link in the email. Defer inbound email parsing indefinitely — the portal is the response interface.

---

## Strategic Decision: Phased Roadmap

### Phase 1 — Complete the Ticket System (Priority: NOW)

> Make "Submit a Question" actually work end-to-end.

| # | Task | Effort |
|---|------|--------|
| 1.1 | **Email notification to consultant** when question is submitted (AWS SES, new template) | 2-3 hrs |
| 1.2 | **Admin Questions Inbox** — new page or section in the portal where consultants see all submitted questions, filtered by engagement/participant | 4-6 hrs |
| 1.3 | **Reply mechanism** — consultant can type a response; response is stored in DB and emailed to participant | 3-4 hrs |
| 1.4 | **Ticket status lifecycle** — Open → In Progress → Resolved, with timestamps | 2-3 hrs |
| 1.5 | **Participant question history** — participant can see their submitted questions and any responses in the wizard or dashboard | 2-3 hrs |
| 1.6 | **Attachment support** — optional screenshot/file on question submission (uses existing S3 pipeline) | 2-3 hrs |

**Total estimated effort:** ~16-22 hours (2-3 working days)

**Outcome:** A fully functional asynchronous support system. Participants ask questions, consultants are notified, respond through the portal, and participants see answers.

### Phase 2 — Email-Enhanced Interactions

> Reduce friction by making email the notification backbone.

| # | Task | Effort |
|---|------|--------|
| 2.1 | **Rich email notification** to consultant — includes question text, participant name, section context, "Respond in Portal" button | 2-3 hrs |
| 2.2 | **Response notification** to participant — email when consultant replies, includes response text + "View in Portal" button | 2-3 hrs |
| 2.3 | **Daily digest** — if multiple questions are pending, send a daily summary email to the engagement lead | 3-4 hrs |

**Total estimated effort:** ~8-10 hours (1-2 working days)

### Phase 3 — Advanced Channels (Future)

> Evaluate based on scale and user feedback.

| # | Channel | Trigger to evaluate |
|---|---------|-------------------|
| 3.1 | **Live Chat** (DevExtreme DxChat) | >50 concurrent participants OR dedicated support person hired |
| 3.2 | **WhatsApp Business API** | Demand from participants + dedicated business phone number |
| 3.3 | **Microsoft Teams integration** | Corporate clients who live in Teams |
| 3.4 | **AI-powered auto-responses** | Sufficient question history to train on patterns |

---

## Data Model Sketch (Phase 1)

### Existing: `support_questions`

```sql
CREATE TABLE support_questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    submission_id UUID NOT NULL REFERENCES submissions(id),
    section_key TEXT NOT NULL,
    subject TEXT NOT NULL,
    message TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'open',     -- open, in_progress, resolved
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### New: `support_replies`

```sql
CREATE TABLE support_replies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_id UUID NOT NULL REFERENCES support_questions(id),
    author_type TEXT NOT NULL,               -- 'consultant' or 'participant'
    author_email TEXT NOT NULL,
    message TEXT NOT NULL,
    attachment_url TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

This creates a threaded conversation model: one `support_question` → many `support_replies`. The consultant and participant can exchange multiple messages on a single question.

## API Sketch (Phase 1)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/participant/submissions/{id}/questions` | Participant JWT | Submit a question (exists) |
| `GET` | `/api/participant/submissions/{id}/questions` | Participant JWT | List my questions + replies (new) |
| `GET` | `/api/engagements/{id}/questions` | Consultant JWT | List all questions for an engagement (new) |
| `POST` | `/api/questions/{id}/replies` | Consultant JWT | Reply to a question (new) |
| `PATCH` | `/api/questions/{id}/status` | Consultant JWT | Update question status (new) |

## UI Sketch (Phase 1)

### Admin: Questions Inbox

Options for placement:
1. **Dedicated page** — "Questions" in the left navigation, similar to Communications
2. **Tab on CommunicationsPage** — add a "Questions" tab alongside the existing outbound communications
3. **Section within EngagementDetail** — questions scoped per engagement

**Recommendation:** Option 1 (dedicated page). Questions are a distinct workflow from outbound communications. A dedicated page with filters (by engagement, participant, status) gives the clearest UX.

### Participant: My Questions

Options for placement:
1. **Badge/counter on the wizard right panel** — "Submit a Question (2)" showing response count
2. **Questions list in the participant dashboard** — below the wizard cards
3. **Modal accessible from the right panel** — "View my questions" link

**Recommendation:** Option 3 for Phase 1 (lightweight modal), evolve to Option 2 for Phase 2.

---

## Open Questions

1. **SLA expectations** — Should we display an expected response time? ("We aim to respond within 24 hours")
2. **Notification frequency** — Should the consultant receive one email per question, or a batched digest?
3. **Question categories** — Should we add a category/type field (clarification, technical issue, content question, other)?
4. **Escalation** — If a question is urgent, should there be a priority flag?
5. **Analytics** — Should we track question patterns to improve the wizard content? (e.g., if Section D generates 80% of questions, the guidance needs improvement)

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-03-09 | Ticket system is the primary channel for Phase 1 | Already partially built; asynchronous nature matches consultant availability; auditable |
| 2026-03-09 | Email notifications complement tickets, not replace them | Email is the delivery mechanism, portal is the interaction surface |
| 2026-03-09 | Live chat deferred to Phase 3 | Scale does not justify infrastructure; unanswered chat is worse UX than a ticket |
| 2026-03-09 | WhatsApp integration deferred indefinitely | High complexity, compliance concerns, manual WhatsApp already works |

---

*Part of [Participant Interactions](./) — [Limitless Portal Design](../../CLAUDE.md)*
