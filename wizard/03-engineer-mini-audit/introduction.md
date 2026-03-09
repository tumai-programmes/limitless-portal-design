# 03 Engineer Mini-Audit — Introduction Step

> **Instrument:** Invincibility Blueprint — Engineer Mini-Audit
> **Audience:** Field engineers (all streams)
> **Position:** New Step 0 (before Consent)
> **Questions:** 3 (all display-only — no interactive elements)
> **Gate:** None — freely navigable to next step

---

## Step 0 — Introduction

| Property | Value |
|----------|-------|
| Step number | 0 of 12 |
| Section | Introduction |
| Questions | 3 (all display-only) |
| Question types | Display only |
| Gate | None — "Save and continue" enabled immediately |

### Question Inventory

| # | Label | Type | Required |
|---|-------|------|----------|
| I.1 | Welcome message | Display only | — |
| I.2 | What to expect | Display only | — |
| I.3 | Key principles | Display only | — |

### Content Blocks

#### I.1 — Welcome

> **Welcome**
>
> Thank you for taking a few minutes to complete this short survey.
>
> Your company is going through an operational diagnostic called the Invincibility Blueprint™. As someone who does the actual work on the ground, your perspective is the most valuable part of this process.
>
> This survey helps identify where time is lost, where rework happens, and what gets in the way of successful jobs. Your answers go directly into building a better way of working.

#### I.2 — What to expect

> **What to Expect**
>
> | | |
> |---|---|
> | **Time** | 10–12 minutes |
> | **Sections** | 10 short sections |
> | **Format** | Mostly short answers and tick-boxes |
> | **Saving** | Your progress is saved — you can come back and finish later |
> | **Anonymous** | You can choose whether to give your name or not |
>
> The questions cover your daily workflow, the tools and portals you use, time sinks, materials, quality, and safety. Some questions are tailored to your job type (installs, service, pre-enablement, or enablement).

#### I.3 — Key principles

> **A Few Things to Know**
>
> - **No blame.** This is about the system, not about you. We want to understand what makes your job harder than it needs to be.
> - **Best estimates are fine.** You don't need exact numbers — your honest feel for things is what matters.
> - **Say it how it is.** The more honest you are, the more useful this is. There are no wrong answers.
> - **It's confidential.** Your individual answers are not shared with management. They are combined with other responses to find patterns.

### Design Notes

- The shortest and most direct of the three intro steps — engineers are completing this on phones between jobs.
- Only 3 content blocks (vs. 5 for Company Audit, 4 for Manager Audit) to keep the entry barrier minimal.
- Language is deliberately plain and conversational — no jargon, no corporate-speak.
- The "anonymous" mention in I.2 pre-empts a common concern and matches the optional name field in Section 1.
- Visual treatment: single guidance card or clean text block — minimal visual complexity for mobile screens.
- The "Save and continue" button is enabled immediately (gating happens at the next step, Consent).
- The progress sidebar shows "Introduction" as the first item, highlighted as active.
- On return visits, the wizard may skip directly to the last incomplete section, but Introduction remains accessible via the sidebar.

### Impact on Step Numbering

With Introduction as the new Step 0:

| New step | Previous step | Section |
|----------|---------------|---------|
| 0 | — (new) | Introduction |
| 1 | 0 | Confidentiality & Consent |
| 2 | 1 | 1 — About You |
| 3 | 2 | 2 — What You Do |
| ... | ... | ... |
| 12 | 11 | Review & Submit |

Total steps changes from **12** (Consent + 11) to **13** (Introduction + Consent + 11).
