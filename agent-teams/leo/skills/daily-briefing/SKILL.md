---
name: daily-briefing
description: >
  Generate and deliver the daily action briefing for the BD team. Pulls AT_RISK
  opportunities and today's due tasks from Twenty CRM, organises by priority, sends to
  Feishu. Runs automatically at 08:00 Taiwan time via cron, or on-demand.
triggers:
  - "daily briefing"
  - "今天有什麼要做"
  - "morning briefing"
version: "3.1"
author: Leo (BD Director Agent)
---

# Daily Briefing

## Purpose

Every morning at 08:00 Taiwan time: read overnight diagnostics from Twenty CRM, pull AT_RISK opportunities and today's due tasks, format into a clean briefing, and send to Feishu.

No computation — just reads already-processed data and formats it. Reads from the `healthCheck` field (written by `deal-progressing` and `deal-health-check`) — not `dealStatus`.

---

## CRM Reference

**Twenty CRM:** `http://localhost:3001` (always localhost)
**GraphQL endpoint:** `http://localhost:3001/graphql`

---

## Data Pull (parallel)

### 1. AT_RISK Opportunities

```graphql
query {
  opportunities(
    filter: {
      and: [
        { healthCheck: { eq: "AT_RISK" } }
        { stage: { neq: "CLOSED_WON" } }
        { stage: { neq: "CLOSED_LOST" } }
      ]
    }
    orderBy: { lastUpdateDate: AscNullsFirst }
  ) {
    edges {
      node {
        id name stage
        lastUpdateDate
        currentStatusSummary
        nextActionSummary
        healthCheck
        company { name }
        owner { name { firstName lastName } }
      }
    }
  }
}
```

### 2. NEEDS_FOLLOWUP Opportunities (secondary tier)

Same query, replace `AT_RISK` with `NEEDS_FOLLOWUP`, and include `currentStatusSummary` and `nextActionSummary` fields.

### 3. Today's Due Tasks

```graphql
query {
  tasks(
    filter: {
      and: [
        { status: { neq: "DONE" } }
        { dueAt: { lte: "{today_end_iso}" } }
      ]
    }
    orderBy: { taskPriority: AscNullsLast }
  ) {
    edges {
      node {
        id
        title { text }
        dueAt
        taskPriority
        status
        assignee { name { firstName lastName } }
      }
    }
  }
}
```

---

## Algorithm

```
1. Fetch AT_RISK opportunities + NEEDS_FOLLOWUP opportunities + today's tasks (parallel)
2. Sort opportunities by lastUpdateDate ascending (oldest = most urgent)
3. Sort tasks by taskPriority (HIGH first)
4. Format briefing
5. Send to Feishu origin chat
```

---

## Message Format

```
🎯 Daily Briefing — {YYYY-MM-DD}

🔴 AT RISK ({n})
• {Company} — {Opportunity Name}
  └ {currentStatusSummary}
  └ Next: {nextActionSummary}
• ...
[if none: No opportunities at risk ✅]

🟡 Needs Follow-up ({n})
• {Company} — {Opportunity Name} — {nextActionSummary}
[if none: omit section]

✅ Tasks Due Today ({n})
• [{priority}] {Task title}
• ...
[if none: No tasks due today]

---
{TIME} · Leo
```

---

## Output Delivery

- **Cron:** Deliver to origin Feishu chat (`deliver: "origin"`)
- **Manual:** Reply in conversation
- **If nothing to report:** Still send — "No opportunities at risk. No tasks due today. ✅" — confirms the briefing ran.

---

## Cron Integration

**Schedule:** `0 8 * * *` (08:00 Taiwan = 00:00 UTC)
**Dependencies:** `daily-deal-health-check` should complete before this (runs at 03:00).
**Delivery:** origin Feishu chat

---

## Pitfalls

1. **Always use localhost** — never external URL.

2. **`amount.amountMicros`** — divide by 1,000,000 to display USD.

3. **`dueAt` filter for "today"** — use `lte: "{today_end_iso}"` to catch overdue tasks too, not just exactly-today.

4. **`title` is RICH_TEXT on tasks** — extract `.text` from the object.

5. **Don't send empty briefing silently** — even if no AT_RISK opportunities and no tasks, send a confirmation message so the team knows the briefing ran.

6. **`taskPriority` options** — `HIGH` / `MEDIUM` / `LOW` (our custom field, not Twenty's built-in status).

7. **Read `healthCheck` field — not `dealStatus` — for opportunity health status.**
