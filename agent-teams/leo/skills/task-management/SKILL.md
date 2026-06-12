---
name: task-management
description: >
  Use when tasks need to be identified, created, or updated in Twenty CRM.
  Automatically called after engagement-logging to extract actionable work items
  from engagement content. Also used when deal-progressing detects stalls.
  Identifies multiple tasks from a single engagement, ensures each has complete
  information (what, who, when), and links them to the correct Opportunity or Partnership.
triggers:
  - "建 task"
  - "幫我記一下要做的事"
  - "有幾件事要做"
  - "新增任務"
  - called automatically from engagement-logging
  - called automatically from deal-progressing
version: "1.0"
author: Leo (BD Director Agent)
---

# Task Management

## Purpose

After an engagement is logged, Leo actively identifies **all actionable work items**
from the engagement content and creates Task records in Twenty CRM. A task is not
just the next step summary — it is a discrete piece of work that someone needs to
execute, with a clear owner, deadline, and enough context to act on.

**Task vs nextAction:**
| | `nextAction` field (on Engagement) | Task (Twenty CRM) |
|---|---|---|
| What it is | One-line summary of the next move | A discrete work item to be executed |
| Format | "{Owner}: {action} by {deadline}" | Full record with title, due date, priority, agent advice |
| How many | Always exactly one per engagement | Zero to many per engagement |
| Who manages | Auto-filled during engagement logging | Tracked, updated, marked DONE |

---

## CRM Reference

**Twenty CRM:** `http://localhost:3001` (always localhost)
**GraphQL endpoint:** `http://localhost:3001/graphql`

### Task Fields
| Field | Twenty field name | Type |
|---|---|---|
| Title | `title` | TEXT (structured as `{ text: "..." }`) |
| Status | `status` | SELECT |
| Due Date | `dueAt` | DATE_TIME |
| Priority | `taskPriority` | SELECT |
| Agent Advice | `agentAdvice` | TEXT |
| Assigned To | `assignees` | RELATION → workspace member |

### `status` options
`TODO` / `IN_PROGRESS` / `DONE`

### `taskPriority` options
`URGENT` / `HIGH` / `MEDIUM` / `LOW`

---

## Step 1: Identify Tasks from Engagement Content

Read the confirmed engagement summary, outcome, notes, and next action. Scan for **any of the following signals** that indicate real work needs to happen:

| Signal type | Examples | Typical task |
|---|---|---|
| Document preparation | "需要報價單", "準備 proposal", "要寄合約", "做個簡報" | Prepare quotation / draft proposal / prepare contract |
| Research / investigation | "要了解他們的技術架構", "查一下他們的預算規模", "研究競爭對手" | Research [topic] before next meeting |
| Follow-up communication | "下週寄 follow-up email", "要回覆他們的問題", "確認一下那個問題" | Draft and send follow-up to [person] |
| Internal coordination | "要跟 Kevin 確認定價", "需要 Hunter 簽 off", "跟技術團隊核一下" | Align with [person] on [topic] |
| Client deliverable | "他們需要 demo 環境", "要準備 POC 資料", "給他們 case study" | Prepare [deliverable] for [company] |
| Deadline / commitment | "他們說下個月要決定", "我們答應週五前給報價" | Deliver [thing] by [date] — committed to client |
| Waiting / monitoring | "等他們內部審批", "等預算確認" | Monitor: follow up if no response by [date+N] |

**Create a task for each distinct work item identified.** One engagement can produce zero tasks (if it was purely informational) or multiple tasks (if several things need to happen).

---

## Step 2: For Each Task — Determine Complete Information

Before creating, fill in:

| Field | How to determine |
|---|---|
| **Title** | Clear, action-oriented: verb + object + context. E.g. "Prepare quotation for ABC Corp — [Product] Standard" |
| **Owner** | Who is responsible? Hunter / Leo / Kevin / client-side person. Infer from engagement content. If unclear, default to Hunter and note it. |
| **Due date** | Any deadline mentioned? If not explicit, infer from deal urgency: AT_RISK → tomorrow, NEEDS_FOLLOWUP → 3 days, ON_TRACK → 7 days |
| **Priority** | URGENT if committed to client or deal is AT_RISK; HIGH if directly blocking deal progression; MEDIUM for preparation tasks; LOW for monitoring tasks |
| **Agent Advice** | Context from the engagement: why this task matters, what to watch for, suggested approach, relevant background |

---

## Step 3: Create Task Records

For each identified task:

```graphql
mutation {
  createTask(data: {
    title: { text: "{action-oriented title}" }
    status: "TODO"
    dueAt: "{deadline_iso}"
    taskPriority: "{priority}"
    agentAdvice: "{context and advice}"
  }) {
    id
    title { text }
    dueAt
    taskPriority
  }
}
```

Then link to Opportunity or Partnership:

```graphql
mutation {
  updateTask(id: "{task_id}", data: {
    taskTargets: {
      connect: { opportunityId: "{opportunity_id}" }
    }
  }) { id }
}
```

If it's a Partnership task, use `partnershipId` instead of `opportunityId`.

---

## Step 4: Check for Stale / Duplicate Tasks

Before creating, check if a similar open task already exists for this Opportunity:

```graphql
query {
  tasks(filter: {
    and: [
      { taskTargets: { opportunity: { id: { eq: "{opportunity_id}" } } } }
      { status: { neq: "DONE" } }
    ]
  }) {
    edges { node { id title { text } dueAt taskPriority } }
  }
}
```

If a task with a very similar title already exists:
- Don't duplicate — update the existing task's due date or priority instead
- Note in agentAdvice: "Updated from engagement on {date}"

---

## Step 5: Mark Completed Tasks

If the engagement notes mention that something was done (e.g. "已經寄了 proposal", "上次說要查的東西查完了"), find and close the corresponding task:

```graphql
mutation {
  updateTask(id: "{task_id}", data: {
    status: "DONE"
  }) { id status }
}
```

---

## Step 6: Stall Task (called from deal-progressing)

When `deal-progressing` determines `healthCheck == AT_RISK`, create a stall task:

```graphql
mutation {
  createTask(data: {
    title: { text: "[STALL] {company_name} — {N} days no activity" }
    status: "TODO"
    dueAt: "{tomorrow_iso}"
    taskPriority: "URGENT"
    agentAdvice: "Deal has gone quiet for {N} days. Last engagement: {last_engagement_summary}. Options: (1) re-engage with a specific question about {last_topic}, (2) escalate to Hunter, (3) assess if deal should be marked lost."
  }) { id }
}
```

Link to Opportunity:
```graphql
mutation {
  updateTask(id: "{task_id}", data: {
    taskTargets: { connect: { opportunityId: "{opportunity_id}" } }
  }) { id }
}
```

---

## Step 7: Report Tasks Created

After all tasks are created, include a task summary in the confirmation to the human:

```
📋 已建立 {N} 個任務：

1. [{priority}] {title} — due {date} ({owner})
   → {one-line agent advice}

2. [{priority}] {title} — due {date} ({owner})
   → {one-line agent advice}
```

If zero tasks were identified, do not mention tasks — keep the confirmation clean.

---

## Verification Checklist

- [ ] Scanned engagement content for all task signals
- [ ] Duplicate / stale tasks checked before creating
- [ ] Each task has: title, status=TODO, dueAt, taskPriority, agentAdvice
- [ ] Each task linked to Opportunity or Partnership
- [ ] Completed tasks (if any mentioned) marked DONE
- [ ] Task summary included in human confirmation

---

## Pitfalls

1. **Don't create vague tasks** — "Follow up" is not a task. "Draft follow-up email to David Lee re: pricing questions by Thursday" is.

2. **Don't duplicate** — check existing open tasks before creating. One Opportunity can accumulate tasks fast; duplicates create noise.

3. **Infer owner from context** — if the engagement says "Hunter said he'd send the contract", owner is Hunter. If Leo can do it (draft email, prepare summary), owner is Leo. Default to Hunter when unclear.

4. **Deadline inference** — if no explicit deadline, use deal health to infer urgency. AT_RISK = tomorrow. Active deal = within the week. Monitoring task = 2 weeks.

5. **Agent advice is not a repeat of the title** — it should add context: why this matters, what to watch, suggested approach. At minimum: one sentence of useful background the human can act on.

6. **`dueAt` format** — ISO 8601: `"2026-06-15T09:00:00.000Z"`.

7. **`title` is structured** — Twenty expects `{ text: "..." }`, not a plain string.
