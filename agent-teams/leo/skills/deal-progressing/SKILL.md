---
name: deal-progressing
description: >
  Use when an Engagement is logged or opportunity state changes. Fetches all Engagements
  for an Opportunity from Twenty CRM, recalculates opportunity health, priority, and risk, then
  updates the Opportunity record. Triggered automatically by engagement-logging,
  or manually when user asks about opportunity status.
triggers:
  - "opportunity progressing"
  - "opportunity status"
  - "update opportunity"
  - "opportunity health"
version: "3.2"
author: Leo (BD Director Agent)
---

# Deal Progressing

## Purpose

After every Engagement is logged, re-evaluate the opportunity and update the Opportunity record in Twenty CRM with a fresh health assessment.

This skill is the PRIMARY health updater — it fires immediately after every engagement is logged. `deal-health-check` is the secondary time-based safety net that runs daily for Opportunities that have gone quiet with no engagement.

---

## CRM Reference

**Twenty CRM:** `http://localhost:3001` (always localhost)
**GraphQL endpoint:** `http://localhost:3001/graphql`

**Opportunity object:** `opportunity` (Twenty internal name)

### Fields Read
| Field | Twenty name |
|---|---|
| Stage | `stage` |
| Engagements | `engagements` (relation) |
| Tasks | `taskTargets` (relation) |

### Fields Written
| Field | Twenty name | Type |
|---|---|---|
| Current Status Summary | `currentStatusSummary` | TEXT |
| Next Action Summary | `nextActionSummary` | TEXT |
| Priority | `priority` | SELECT |
| Health Check | `healthCheck` | SELECT |
| Last Person Contact Date | `lastUpdateDate` | DATE_TIME |

### `priority` options: `VERY_HIGH` / `HIGH` / `MEDIUM` / `LOW`
### `healthCheck` options: `ON_TRACK` / `NEEDS_FOLLOWUP` / `AWAITING` / `AT_RISK`

---

## Workflow

### Step 1: Fetch Opportunity + Engagement History

```graphql
query GetOpportunityWithEngagements($id: ID!) {
  opportunity(id: $id) {
    id
    name
    stage
    amount { amountMicros currencyCode }
    closeDate
    currentStatusSummary
    healthCheck
    priority
    lastUpdateDate
    engagements {
      edges {
        node {
          id
          engagementDate
          engagementType
          engagementStatus
          outcome
          engagementNote { json }
          nextAction
        }
      }
    }
    taskOpportunities {
      edges {
        node {
          id
          title { text }
          status
          dueAt
        }
      }
    }
  }
}
```

---

### Step 2: Analyse Signals

From all **Completed** engagements:
```
signals:
  total_engagements     = count(status == COMPLETED)
  last_engagement_date  = max(engagementDate)
  days_since_last       = (today - last_engagement_date).days
  outcomes              = list of outcome texts
  open_tasks            = count(tasks where status != DONE)
  overdue_tasks         = count(tasks where dueAt < today and status != DONE)
  current_stage         = opportunity.stage
```

---

### Step 3: Determine Health Check

```
if days_since_last > 14:
    healthCheck = AT_RISK
elif days_since_last > 7:
    healthCheck = NEEDS_FOLLOWUP
elif overdue_tasks > 0:
    healthCheck = NEEDS_FOLLOWUP
elif any negative signal in recent outcomes:
    healthCheck = AWAITING
else:
    healthCheck = ON_TRACK
```

---

### Step 4: Determine Priority

```
if stage in [D3, D4, S1, S2]:        priority = VERY_HIGH
elif stage == D2 and ON_TRACK:        priority = HIGH
elif healthCheck == AT_RISK:          priority = VERY_HIGH  (override)
elif days_since_last <= 3:            priority = HIGH
elif days_since_last <= 7:            priority = MEDIUM
else:                                 priority = LOW
```

**Stage reference:**
- D1 = Lead / D2 = Qualified / D3 = Proposal / D4 = Negotiation
- S1 = Contract Review / S2 = Final Sign-off
- CLOSED_WON / CLOSED_LOST = terminal, do not update

---

### Step 5: Generate Summaries

**Current Status Summary** — plain English, one sentence:
```
"{Stage} stage. Last engagement {N} days ago ({type}). {1 key signal from outcomes}."
```
Examples:
- "D3 stage. Last engagement 2 days ago (Online Meeting). Client reviewing proposal."
- "D2 stage. No activity for 9 days. Awaiting budget approval."

**Next Action Summary** — one actionable sentence:
```
if AT_RISK:     "Re-engage immediately — call or email to assess opportunity health."
if NEEDS_FOLLOWUP: "Follow up on {last_next_action_from_engagement}."
if AWAITING:    "Wait for client response by {expected_date}. Follow up if no reply by {date+3}."
if ON_TRACK:    "{last_next_action_from_engagement or 'Continue opportunity progression.'}"
```

---

### Step 6: Update Opportunity

```graphql
mutation UpdateOpportunity($id: ID!, $input: UpdateOneOpportunityInput!) {
  updateOpportunity(id: $id, input: $input) {
    id
    healthCheck
    priority
    currentStatusSummary
    nextActionSummary
    lastUpdateDate
  }
}
```

Variables:
```json
{
  "id": "{opportunity_id}",
  "input": {
    "opportunity": {
      "currentStatusSummary": "{summary}",
      "nextActionSummary": "{next_action}",
      "priority": "{priority}",
      "healthCheck": "{health_check}",
      "lastUpdateDate": "{now_iso}"
    }
  }
}
```

---

### Step 7: Create Stall Task (if AT_RISK)

If `healthCheck == AT_RISK`, auto-create a re-engagement task:

```graphql
mutation {
  createTask(data: {
    title: { text: "[STALL] {company_name} — {days} days no activity" }
    status: "TODO"
    dueAt: "{tomorrow_iso}"
    taskPriority: "URGENT"
    agentAdvice: "Opportunity has gone quiet for {days} days. Review last engagement notes and choose: (1) re-engage with a specific question, (2) escalate to [Sales Rep], (3) mark as lost."
  }) { id }
}
```

Then link the task to the opportunity via `taskTargets`:

```graphql
mutation {
  updateTask(id: "{task_id}", data: {
    taskTargets: { connect: { opportunityId: "{opportunity_id}" } }
  }) { id }
}
```

---

## Daily Cron Mode

When triggered by `daily-deal-health-check` cron:
1. Fetch all open opportunities (stage ≠ CLOSED_WON / CLOSED_LOST)
2. For each opportunity, run Steps 1–7
3. Report: opportunities flagged AT_RISK, tasks created, opportunities updated

---

## Pitfalls

1. **Always use localhost** — never external URL.

2. **Skip closed opportunities** — CLOSED_WON and CLOSED_LOST are terminal states. Do not update priority or health on them.

3. **`lastUpdateDate` format** — ISO 8601: `"2026-06-11T08:00:00.000Z"`.

4. **RICH_TEXT fields** — `engagementNote` is RICH_TEXT. Extract the plain text from `.json` before processing sentiment.

5. **Only count Completed engagements for health analysis** — Planned engagements don't indicate interaction has occurred.

6. **`taskOpportunities` is the back-relation name** — tasks linked via `opportunity` relation on the task side appear here. Confirm field name if query returns empty.
