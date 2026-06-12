---
name: deal-health-check
description: >
  Time-based daily scan of all active Opportunities. Detects opportunities that have gone
  quiet — no engagement logged recently — and updates their healthCheck field accordingly.
  Complements deal-progressing (which fires on each engagement logged). This skill is the
  safety net; deal-progressing is the primary health updater. Runs autonomously as a cron
  trigger or can be called manually. Use when: cron triggers at 11:00 daily, or user asks
  "opportunity 健檢", "哪些 opportunity 卡住了", "pipeline 有沒有 at risk".
triggers:
  - "opportunity health check"
  - "opportunity 健檢"
  - "哪些 opportunity 卡住了"
  - "pipeline 有沒有 at risk"
  - "AT_RISK opportunities"
version: "2.0"
author: BD Lead Agent
---

# Opportunity Health Check

## Purpose
Time-based daily scan of all active Opportunities. Complements deal-progressing (which fires on each engagement). This skill catches Opportunities that have gone quiet — no engagement logged recently — and updates their healthCheck accordingly. deal-progressing is the primary health updater; this skill is the safety net.

HealthCheck is the single source of truth. Both this skill and deal-progressing write to the same field.

## healthCheck values (unified with deal-progressing)
- **ON_TRACK**: engaged within 7 days, no overdue tasks
- **NEEDS_FOLLOWUP**: 7–14 days no engagement, or overdue tasks
- **AT_RISK**: 14+ days no engagement
- **AWAITING**: waiting for client response (set by deal-progressing, not by this skill)

## CRM Reference
Twenty CRM: http://localhost:3001 (always localhost)
GraphQL endpoint: http://localhost:3001/graphql

## Step 1 — Fetch All Active Opportunities with Engagement History
Query all non-closed opportunities, including their most recent completed engagement date:

```graphql
query ActiveOpportunitiesWithEngagement {
  opportunities(
    filter: {
      and: [
        { stage: { notIn: ["CLOSED_WON", "CLOSED_LOST"] } }
      ]
    }
    first: 100
  ) {
    edges {
      node {
        id
        name
        stage
        healthCheck
        currentStatusSummary
        nextActionSummary
        lastUpdateDate
        updatedAt
        closeDate
        amount { amountMicros currencyCode }
        company { name }
        pointOfContact { edges { node { id name { firstName lastName } } } }
        engagements(
          filter: { engagementStatus: { eq: "COMPLETED" } }
          orderBy: { engagementDate: DescNullsLast }
          first: 1
        ) {
          edges { node { engagementDate engagementType outcome nextAction } }
        }
        taskOpportunities(
          filter: { status: { notIn: ["DONE"] } }
        ) {
          edges { node { id title { text } dueAt taskPriority status } }
        }
      }
    }
  }
}
```

## Step 2 — Calculate Days Since Last Engagement
For each opportunity:
- `last_engagement_date` = most recent completed engagement date (from query)
- If no engagement ever: use `createdAt` as baseline
- `days_since_engagement` = (today − last_engagement_date).days
- `overdue_tasks` = count(tasks where dueAt < today and status != DONE)

## Step 3 — Apply Time-Based Health Rules
Only update healthCheck if the current value was set by this skill (time-based), NOT if it was set by deal-progressing after a recent engagement. The rule: if last_engagement_date is within the last 3 days, skip this opportunity — deal-progressing already has fresh data.

```
if days_since_engagement <= 3:
    skip — deal-progressing has fresh data, don't override
elif days_since_engagement > 14:
    new_healthCheck = AT_RISK
elif days_since_engagement > 7 or overdue_tasks > 0:
    new_healthCheck = NEEDS_FOLLOWUP
else:
    new_healthCheck = ON_TRACK
```

If current `healthCheck == AWAITING`: do not override. AWAITING is set by deal-progressing when human explicitly says client is reviewing/deciding.

## Step 4 — Update healthCheck (only if changed)
Only write to CRM if the healthCheck value is changing:

```graphql
mutation {
  updateOpportunity(
    id: "{opportunity_id}"
    data: {
      healthCheck: "{new_healthCheck}"
      currentStatusSummary: "{stage} stage. No engagement for {N} days. {last_outcome_if_known}"
      nextActionSummary: "{appropriate_next_action}"
      lastUpdateDate: "{today_iso}"
    }
  ) {
    id
    healthCheck
    currentStatusSummary
  }
}
```

**nextActionSummary logic:**
- **AT_RISK**: `"Re-engage immediately — {N} days of silence. Last interaction: {date}."`
- **NEEDS_FOLLOWUP**: `"Follow up on: {last nextAction from engagement if known, else 'check in on status'}"`
- **ON_TRACK**: keep existing nextActionSummary if set, otherwise `"Continue progression."`

## Step 5 — Create Stall Task (if AT_RISK and no recent stall task)
Before creating, check if a stall task already exists (open, created in last 7 days):

```graphql
query {
  tasks(
    filter: {
      and: [
        { taskTargets: { opportunity: { id: { eq: "{opportunity_id}" } } } }
        { status: { notIn: ["DONE"] } }
        { title: { like: "%[STALL]%" } }
      ]
    }
  ) {
    edges { node { id } }
  }
}
```

If no existing stall task:

```graphql
mutation {
  createTask(data: {
    title: { text: "[STALL] {company_name} — {N} days no engagement" }
    status: "TODO"
    dueAt: "{tomorrow_iso}"
    taskPriority: "URGENT"
    agentAdvice: "Opportunity has gone quiet for {N} days. Last engagement: {date, type}. Options: (1) re-engage with a specific question about {last topic}, (2) escalate to [Sales Rep], (3) assess if opportunity should be marked lost."
  }) { id }
}
```

Then link to opportunity via taskTargets.

## Step 6 — Output Summary
Return structured output for daily-briefing to consume:

```
## Opportunity Health Scan — {DATE}

Scanned: {N} active opportunities
Skipped (fresh engagement ≤3 days): {N}
Updated: {N}

### 🔴 AT_RISK ({n})
| Opportunity | Company | Stage | Days silent | Last engagement | Next action |
|---|---|---|---|---|---|
| {name} | {company} | {stage} | {N}d | {date, type} | {nextActionSummary} |

### 🟡 NEEDS_FOLLOWUP ({n})
| Opportunity | Company | Stage | Days silent | Next action |
|---|---|---|---|---|

### ✅ ON_TRACK ({n})
(names only)
```

**Mode B (cron):** Silent if all opportunities are ON_TRACK and no updates were made.
**Mode A (manual, single opportunity):** Always return full detail regardless.

## Pitfalls
1. Always use localhost — never external URL
2. Do NOT use `dealStatus` field — `healthCheck` is the single source of truth
3. Do NOT override AWAITING — that status is set by deal-progressing when human reports client is deciding
4. Do NOT override if last engagement ≤ 3 days — deal-progressing has fresh data
5. Always dedup stall tasks — check before creating
6. `lastUpdateDate` may be null — fall back to `updatedAt`, then `createdAt`
7. Stage confirmed values: `NEW`, `SCREENING`, `MEETING`, `PROPOSAL`, `CUSTOMER`
8. `engagementNote` is RICH_TEXT — extract plain text from `.json`
