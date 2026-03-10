# Phase 1 Action Plan — Ticket System

> **Status:** Draft — ready for review
> **Created:** 9 March 2026
> **Depends on:** `strategy.md` (channel decisions)
> **Delivers to:** `limitless-portal` (code repo)

---

## Objective

Make "Submit a Question" work end-to-end: participant asks → consultant is notified → consultant views and responds → participant sees the response.

---

## UI Design — The Core Challenge

### The Problem

A support question has three dimensions of context:
1. **Who** — the participant (name, role, engagement)
2. **Where** — the wizard section they were in when they asked
3. **What** — the question thread (messages back and forth)

Showing all three simultaneously in one view creates visual overload. The question is: which dimension becomes the primary navigation axis?

### Industry Patterns Considered

| Pattern | Primary axis | Example | Fit |
|---------|-------------|---------|-----|
| **Inbox/Conversation** | Time-sorted thread list | Intercom, Zendesk, Gmail | Best for async support |
| **Kanban board** | Status columns (open/in-progress/resolved) | Jira, Trello | Over-engineered for <50 tickets |
| **Embedded comments** | Location in document | Google Docs, Notion | Requires the consultant to navigate through the wizard |
| **Simple list + detail** | Flat table with expand/modal | GitHub Issues | Clean, low complexity |

### Recommended Pattern: **Inbox with Context Panel**

This is the Intercom/Zendesk pattern adapted for our scale. It separates the three dimensions into distinct spatial zones:

```
┌─────────────────────────────────────────────────────────────────────┐
│ [←] Questions                                              [Filter]│
├──────────────────┬──────────────────────────┬───────────────────────┤
│                  │                          │                       │
│  QUESTION LIST   │  CONVERSATION THREAD     │  CONTEXT PANEL        │
│  (scrollable)    │  (scrollable)            │  (fixed)              │
│                  │                          │                       │
│  ┌────────────┐  │  ┌──────────────────┐    │  PARTICIPANT          │
│  │ ● Open     │  │  │ Mathew Field     │    │  Mathew Field         │
│  │ Mathew F.  │◄─┤  │ 9 Mar, 10:23    │    │  mathew@nano.co.uk    │
│  │ Section 2  │  │  │                  │    │  Role: Engineer       │
│  │ 2h ago     │  │  │ "I'm not sure   │    │  Dept: Installations  │
│  └────────────┘  │  │  what to put for │    │                       │
│  ┌────────────┐  │  │  NPT — is this   │    │  SECTION CONTEXT      │
│  │ ○ Resolved │  │  │  just downtime?" │    │  03-engineer-mini     │
│  │ Sarah T.   │  │  └──────────────────┘    │  Section 3: NPT       │
│  │ Section A  │  │                          │  Progress: 25%        │
│  │ 1d ago     │  │  ┌──────────────────┐    │                       │
│  └────────────┘  │  │ Greg (consultant)│    │  SUBMISSION           │
│                  │  │ 9 Mar, 11:45    │    │  Started: 8 Mar       │
│                  │  │                  │    │  Status: In Progress   │
│                  │  │ "NPT = Non-     │    │  Deadline: 15 Mar     │
│                  │  │  Productive Time │    │                       │
│                  │  │  — any time your │    │  ─────────────────    │
│                  │  │  team can't work │    │  [View Submission →]  │
│                  │  │  due to external │    │                       │
│                  │  │  factors..."     │    │                       │
│                  │  └──────────────────┘    │                       │
│                  │                          │                       │
│                  │  ┌──────────────────┐    │                       │
│                  │  │ [Type reply...]  │    │                       │
│                  │  │           [Send] │    │                       │
│                  │  └──────────────────┘    │                       │
├──────────────────┴──────────────────────────┴───────────────────────┤
│ 2 open · 1 resolved                                                 │
└─────────────────────────────────────────────────────────────────────┘
```

**Why this works:**

1. **Left panel** — the "inbox" — answers "which questions need my attention?" Sorted by newest, with status indicators (open = bold, resolved = muted). This is where the consultant starts.

2. **Centre panel** — the "thread" — answers "what was the conversation?" The consultant reads the exchange and types a reply at the bottom. This is where the consultant works.

3. **Right panel** — the "context" — answers "who is this person and where were they?" The section name, instrument type, submission progress, and participant details are always visible without cluttering the thread. This is where the consultant glances.

The key insight: **the conversation is the primary object, and the section context is metadata displayed alongside it — not a navigation axis.** The consultant does not need to navigate to "Section 2" and then find questions there. They see all questions in a flat inbox, and the section context appears when they click one.

### Why NOT a Separate Page Per Section

The alternative — grouping questions by section — forces the consultant to think "which section might have questions?" instead of simply scanning an inbox. With <50 participants, the question volume is low enough that a flat, time-sorted list is the most efficient. Section context is shown as a label and in the right panel — no need to make it a navigation dimension.

---

## Notification Badge in Zone (A)

The screenshot shows the wizard top bar. Zone (A) — the area around the avatar/user menu — is the ideal place for a notification indicator.

### For Participants (Wizard TopBar)

```
┌──────────────────────────────────────────────────────────────────┐
│ LIMITLESS MODUS | nano   participant > NANO FIBRE  ● Saved  🔔² │
│                                                     M. Field ▼  │
└──────────────────────────────────────────────────────────────────┘
                                                      ▲
                                                      │
                                            Notification bell
                                            with unread count
```

- A bell icon with a badge showing unread reply count
- Clicking opens a small dropdown: "You have 2 new replies" with a link to each
- Links open a `MyQuestionsModal` showing the full thread
- Badge disappears once the participant views the replies

### For Consultants (Sidebar Nav)

```
┌──────────────┐
│ 🏠 Dashboard │
│ ☑ Audits     │
│ 💬 Interviews│
│ 🔍 Findings  │
│ 📧 Comms     │
│ ❓ Questions²│  ← Badge with count of open/unread questions
│ ⚙ Settings   │
└──────────────┘
```

- "Questions" added to the sidebar navigation with a badge showing open question count
- The badge uses a small red dot or number pill (consistent with how other apps show unread items)
- Badge updates on page navigation (polling) — no need for WebSocket in Phase 1

---

## Participant-Side UI Design

### "Submit a Question" — Already Implemented

The right panel "Submit a Question" button opens `SubmitQuestionModal.vue`. This is fine. No changes needed except:

1. Add the notification bell to `WizardTopBar.vue`
2. Add a "My Questions" link/button to the right panel (below "Submit a Question")

### "My Questions" Modal

When a participant has submitted questions, a new link appears in the right panel:

```
┌──────────────────────────────────────┐
│ TIPS                                 │
│ Answer as best you can...            │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │     Submit a Question            │ │
│ └──────────────────────────────────┘ │
│ ┌──────────────────────────────────┐ │
│ │     My Questions (2)             │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

Clicking "My Questions" opens a DxPopup modal:

```
┌──────────────────────────────────────────────────────┐
│ My Questions                                      ✕ │
├──────────────────────────────────────────────────────┤
│                                                      │
│ ┌──────────────────────────────────────────────────┐ │
│ │ Question about Section 3: NPT           ● Open  │ │
│ │ 9 Mar 2026, 10:23                               │ │
│ │                                                  │ │
│ │ You: "I'm not sure what to put for NPT —        │ │
│ │ is this just downtime?"                          │ │
│ │                                                  │ │
│ │ ┌─ Reply from Greg, 9 Mar 11:45 ─────────────┐  │ │
│ │ │ "NPT = Non-Productive Time — any time your  │  │ │
│ │ │ team can't work due to external factors like │  │ │
│ │ │ access issues, missing materials, weather,   │  │ │
│ │ │ or waiting for other trades..."              │  │ │
│ │ └─────────────────────────────────────────────┘  │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ ┌──────────────────────────────────────────────────┐ │
│ │ Question about Section 2: What You Do    ✓ Done  │ │
│ │ 8 Mar 2026, 15:10                               │ │
│ │ (Collapsed — click to expand)                    │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Interaction:**
- Questions are listed newest-first
- Open questions with replies are expanded by default
- Resolved questions are collapsed
- Participant can submit a follow-up reply within the thread
- Badge clears when the participant opens this modal

---

## Consultant-Side UI Design — Detailed Wireframes

### Questions Page (New)

**Route:** `/questions?engagement=<id>`
**Navigation:** Added to sidebar as "Questions" between "Communications" and "Settings"

#### Empty State

```
┌─────────────────────────────────────────────────────────────────┐
│ Questions                                                       │
│                                                                 │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                                                             │ │
│ │        No questions yet.                                    │ │
│ │                                                             │ │
│ │        When participants submit questions from the          │ │
│ │        wizard, they will appear here.                       │ │
│ │                                                             │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### With Questions (Three-Panel Inbox)

```
┌─────────────────────────────────────────────────────────────────┐
│ Questions                                    [All ▼] [Open ▼]  │
├───────────────┬─────────────────────────────┬───────────────────┤
│ INBOX         │ THREAD                      │ CONTEXT           │
│               │                             │                   │
│ ● Mathew F.  │ Mathew Field                │ PARTICIPANT       │
│   NPT quest… │ 9 Mar, 10:23               │ Mathew Field      │
│   2h ago     │                             │ mathew@nano…      │
│   ──────────  │ "I'm not sure what to put  │ Engineer          │
│ ○ Sarah T.   │  for NPT — is this just    │ Installations     │
│   Org struct… │  downtime?"                │                   │
│   1d ago     │                             │ SECTION           │
│               │ ─────────────────────────── │ 03-engineer-mini  │
│               │                             │ 3: NPT            │
│               │ Greg Kurnikov               │                   │
│               │ 9 Mar, 11:45               │ SUBMISSION        │
│               │                             │ Progress: 25%     │
│               │ "NPT = Non-Productive Time │ Status: started   │
│               │  — any time your team can't│ Due: 15 Mar       │
│               │  work due to external      │                   │
│               │  factors like access       │ ──────────────    │
│               │  issues, missing materials,│ [View Submission] │
│               │  weather, or waiting for   │ [View in Comms]   │
│               │  other trades."            │                   │
│               │                             │                   │
│               │ ┌─────────────────────────┐ │                   │
│               │ │ Type your reply...      │ │                   │
│               │ │                         │ │                   │
│               │ │          [Send] [Close] │ │                   │
│               │ └─────────────────────────┘ │                   │
├───────────────┴─────────────────────────────┴───────────────────┤
│ 1 open · 1 resolved                                             │
└─────────────────────────────────────────────────────────────────┘
```

**Responsive behaviour:**
- ≥1280px: Three-panel layout (inbox 240px + thread flex + context 280px)
- 1024–1279px: Two-panel (inbox + thread); context collapses to a top bar within the thread
- <1024px: Single panel — list view; clicking opens full-screen thread

---

## Implementation Steps

### Step 1: Database — Add `support_replies` table

**File:** `limitless-portal/migrations/011_support_replies.up.sql`

```sql
CREATE TABLE support_replies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_id UUID NOT NULL REFERENCES support_questions(id) ON DELETE CASCADE,
    author_type TEXT NOT NULL CHECK (author_type IN ('consultant', 'participant')),
    author_email TEXT NOT NULL,
    author_name TEXT NOT NULL DEFAULT '',
    message TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_support_replies_question ON support_replies(question_id);
```

Also add status tracking to the existing table:

```sql
ALTER TABLE support_questions
    ADD COLUMN IF NOT EXISTS status TEXT NOT NULL DEFAULT 'open',
    ADD COLUMN IF NOT EXISTS updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW();
```

**Down migration:** `011_support_replies.down.sql`

```sql
DROP TABLE IF EXISTS support_replies;
ALTER TABLE support_questions DROP COLUMN IF EXISTS status;
ALTER TABLE support_questions DROP COLUMN IF EXISTS updated_at;
```

**Effort:** 30 min

---

### Step 2: Backend — Models

**File:** `limitless-portal/internal/models/notification.go` (or new `question.go`)

Add structs:
- `SupportQuestion` — extended with `Status`, `UpdatedAt`, `ParticipantName`, `ParticipantEmail`, `InstrumentKey`, `SectionLabel`, `SubmissionProgress`, `ReplyCount`
- `SupportReply` — `ID`, `QuestionID`, `AuthorType`, `AuthorEmail`, `AuthorName`, `Message`, `CreatedAt`
- `QuestionWithReplies` — `SupportQuestion` + `[]SupportReply`

**Effort:** 30 min

---

### Step 3: Backend — Storage (Repository)

**File:** `limitless-portal/internal/storage/postgres.go` + `repository.go`

Add methods:
- `GetQuestionsByEngagement(engagementID string) ([]SupportQuestion, error)` — joins with submissions and participants to get context
- `GetQuestionWithReplies(questionID string) (*QuestionWithReplies, error)` — single question with all replies
- `CreateSupportReply(reply *SupportReply) error`
- `UpdateQuestionStatus(questionID, status string) error`
- `GetQuestionsBySubmission(submissionID string) ([]SupportQuestion, error)` — for participant view
- `GetUnreadQuestionCount(engagementID string) (int, error)` — for badge
- `GetParticipantUnreadCount(submissionID string) (int, error)` — for participant badge

**Effort:** 2-3 hours

---

### Step 4: Backend — API Handlers

**File:** `limitless-portal/internal/handlers/questions.go` (new file)

| Method | Path | Auth | Handler | Description |
|--------|------|------|---------|-------------|
| `GET` | `/api/engagements/{id}/questions` | Consultant | `GetEngagementQuestions` | List all questions with context |
| `GET` | `/api/questions/{id}` | Consultant | `GetQuestionDetail` | Single question with replies |
| `POST` | `/api/questions/{id}/replies` | Consultant | `CreateReply` | Consultant replies |
| `PATCH` | `/api/questions/{id}/status` | Consultant | `UpdateStatus` | Mark open/resolved |
| `GET` | `/api/participant/submissions/{id}/questions` | Participant | `GetMyQuestions` | Participant's own questions with replies |
| `POST` | `/api/participant/questions/{id}/replies` | Participant | `CreateParticipantReply` | Participant follow-up reply |
| `GET` | `/api/participant/submissions/{id}/unread-count` | Participant | `GetUnreadCount` | Badge count |
| `GET` | `/api/engagements/{id}/questions/count` | Consultant | `GetQuestionCount` | Badge count |

**Effort:** 3-4 hours

---

### Step 5: Backend — Email Notifications

**File:** `limitless-portal/internal/services/email.go` + new template

When a participant submits a question:
1. Store in DB (already works)
2. Send email to engagement admin via AWS SES

**Email template:** `internal/services/templates/support-question-notification.html`

```
Subject: New question from [Participant Name] — [Section Label]

[Participant Name] has submitted a question while completing [Instrument Name].

Section: [Section Label]
Subject: [Question Subject]

"[Question Message]"

[Respond in Portal →]  (links to /questions?engagement=X&question=Y)
```

When a consultant replies:
1. Store reply in DB
2. Send email to participant via AWS SES

**Email template:** `internal/services/templates/question-reply-notification.html`

```
Subject: Reply to your question — [Subject]

Hi [Participant Name],

The engagement team has responded to your question about [Section Label]:

"[Reply Message]"

[View in Portal →]  (links to participant wizard with ?show_questions=true)
```

**Effort:** 2-3 hours

---

### Step 6: Frontend — Consultant Questions Page

**File:** `limitless-portal/frontend/src/views/QuestionsPage.vue` (new)

Three-panel inbox layout as wireframed above:
- Left: DxList or custom list with question summaries
- Centre: Thread view with reply input at bottom
- Right: Context panel with participant + section + submission metadata

Components:
- `QuestionsPage.vue` — main page, data fetching, layout
- `QuestionInboxItem.vue` — single item in the left list
- `QuestionThread.vue` — thread of messages + reply input
- `QuestionContextPanel.vue` — right-side metadata

**Effort:** 4-6 hours

---

### Step 7: Frontend — Navigation and Routing

**File:** `limitless-portal/frontend/src/navigation.ts`

Add "Questions" to sidebar between "Communications" and "Settings":

```typescript
{
  text: 'Questions',
  path: '/questions',
  icon: 'help'  // or 'message'
}
```

**File:** `limitless-portal/frontend/src/router/index.ts`

Add route:

```typescript
{
  path: '/questions',
  name: 'Questions',
  meta: { requiresAuth: true, consultantOnly: true, layout: SideNavOuterToolbar },
  component: () => import('@/views/QuestionsPage.vue')
}
```

**Effort:** 15 min

---

### Step 8: Frontend — Notification Badge (Consultant Sidebar)

**File:** `limitless-portal/frontend/src/components/SideNavMenu.vue`

Add a badge component next to the "Questions" nav item showing the count of open questions. Poll `GET /api/engagements/{id}/questions/count` every 60 seconds.

**Effort:** 1-2 hours

---

### Step 9: Frontend — Notification Badge (Participant TopBar)

**File:** `limitless-portal/frontend/src/components/wizard/WizardTopBar.vue`

Add a bell icon with unread badge in zone (A), between the save status and user menu:

```
.wizard-topbar-right:
  [save status] [bell icon + badge] [user menu]
```

- Bell icon: `dx-icon-bell` or custom SVG
- Badge: small red circle with count (CSS `::after` pseudo-element or positioned `<span>`)
- Click: opens a dropdown with unread reply summaries
- Each summary links to the `MyQuestionsModal`
- Poll `GET /api/participant/submissions/{id}/unread-count` every 60 seconds

**Effort:** 2-3 hours

---

### Step 10: Frontend — "My Questions" (Participant)

**Files:**
- `limitless-portal/frontend/src/components/wizard/MyQuestionsModal.vue` (new)
- `limitless-portal/frontend/src/components/wizard/WizardRightPanel.vue` (modify)

Add "My Questions (N)" button below "Submit a Question" in the right panel. Opens a DxPopup showing all the participant's questions with replies and follow-up input.

**Effort:** 2-3 hours

---

## Summary Checklist

| # | Task | Type | Effort | Depends on |
|---|------|------|--------|-----------|
| 1 | Database migration — `support_replies` + status column | Backend | 30 min | — |
| 2 | Models — `SupportQuestion`, `SupportReply`, `QuestionWithReplies` | Backend | 30 min | 1 |
| 3 | Storage — repository methods | Backend | 2-3 hrs | 2 |
| 4 | API handlers — `questions.go` | Backend | 3-4 hrs | 3 |
| 5 | Email notifications — question submitted + reply sent | Backend | 2-3 hrs | 4 |
| 6 | Questions Page — three-panel inbox | Frontend | 4-6 hrs | 4 |
| 7 | Navigation + routing — add "Questions" | Frontend | 15 min | 6 |
| 8 | Badge — consultant sidebar | Frontend | 1-2 hrs | 4 |
| 9 | Badge — participant topbar (zone A) | Frontend | 2-3 hrs | 4 |
| 10 | My Questions modal — participant view + follow-up reply | Frontend | 2-3 hrs | 4 |

**Total estimated effort:** ~18-26 hours (~3-4 working days)

**Critical path:** Steps 1-4 (backend) must be done first. Steps 5-10 (frontend + email) can be parallelised.

---

## Sequencing Recommendation

```
Day 1:  Steps 1-3 (database + models + storage)
Day 2:  Step 4 (API handlers) + Step 5 (email notifications)
Day 3:  Step 6 (Questions page) + Step 7 (nav/routing)
Day 4:  Steps 8-10 (badges + participant view)
```

---

## Open Questions for Review

1. **Should "Questions" and "Communications" eventually merge?** They are related but serve different directions (inbound vs outbound). Keeping them separate for Phase 1 is cleaner. A future "Conversations" page could unify both.

2. **Should the consultant be able to initiate a message to a participant?** Currently only participants can start a thread. Phase 2 could add a "Message participant" button on the Communications page.

3. **Reply notification email — immediate or batched?** Immediate is simpler and better UX for the participant. Batching only makes sense for the consultant receiving many questions.

4. **Should we track "read" status?** Marking a question as "read" when the consultant views it is different from marking it "resolved." Both are useful. Read status drives the badge; resolved status drives the workflow.

5. **Attachment support?** The design spec mentions optional screenshot on question submission. Defer to a fast follow-up after the core flow works — it uses the existing S3 pipeline.

---

*Part of [Participant Interactions](./) — [Limitless Portal Design](../../CLAUDE.md)*
