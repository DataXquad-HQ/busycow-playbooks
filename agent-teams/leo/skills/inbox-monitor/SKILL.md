---
name: inbox-monitor
version: 1.0
author: BD Lead Agent
description: >
  Monitor the agent's OpenMail inbox for inbound replies. Match replies against
  known outreach targets in Twenty CRM. Notify [Sales Rep] with context and
  suggested next action. Update CRM status on confirmed match.
  Triggered by cron (every 30 minutes) or manually ("check inbox", "any replies?").
triggers:
  - check inbox
  - any replies
  - 有沒有回信
  - 收件箱
  - inbox check
---

## Purpose

Scan the OpenMail inbox for unread inbound messages. For each reply:
1. Identify whether the sender is a known Person in Twenty CRM
2. Classify the reply intent (positive / info request / not now / unsubscribe)
3. Notify [Sales Rep] with full context + suggested next action
4. Update CRM: accountStatus → WARM, log engagement

---

## Credentials

- **API Key**: `~/.hermes/profiles/leo/secrets/openmail_api_key`
- **Inbox ID**: `{{OPENMAIL_INBOX_ID}}`
- **From address**: `{{OPENMAIL_ADDRESS}}`

```bash
OPENMAIL_API_KEY=*** ~/.hermes/profiles/leo/secrets/openmail_api_key)
OPENMAIL_INBOX_ID="{{OPENMAIL_INBOX_ID}}"
```

---

## Workflow

### Step 1: Fetch Unread Inbound Messages

```bash
curl -s "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/threads?is_read=false" \
  -H "Authorization: Bearer ${OPEN...
```

If response is empty (`"data": []`) → no new replies. Report "No new replies" and stop.

For each unread thread, get messages:
```bash
curl -s "https://api.openmail.sh/v1/threads/{thread_id}/messages" \
  -H "Authorization: Bearer ${OPEN...
```

Extract from each inbound message:
- `from` — sender email address
- `subject`
- `body_text` — plain text body
- `threadId` — for future replies

---

### Step 2: Match Sender to CRM

For each sender email, query Twenty CRM:

```graphql
query {
  people(
    filter: {
      emails: { primaryEmail: { eq: "{sender_email}" } }
    }
  ) {
    edges {
      node {
        id
        name { firstName lastName }
        jobTitle
        company {
          id name accountStatus
        }
        pointOfContactForOpportunities {
          edges { node { id name stage } }
        }
      }
    }
  }
}
```

**Match found** → proceed to Step 3
**No match** → flag as unknown sender (Step 3b)

---

### Step 3a: Known Sender — Classify & Notify

Classify reply intent from `body_text`:

| Signal | Classification |
|---|---|
| Asks for more info, meeting, demo, pricing | **Positive interest** |
| "Not now", "maybe later", "busy this quarter" | **Not now** |
| "Wrong person", "not relevant", asks to be removed | **Unsubscribe** |
| Anything else / ambiguous | **Unclear — needs human read** |

**Deliver notification to [Sales Rep]:**
```
📬 Reply received — {Company Name} / {Person Name}

From: {email}
Subject: {subject}
Intent: {Positive interest / Not now / Unsubscribe / Unclear}

--- Message ---
{body_text (truncated to 300 chars)}
---

Suggested next action:
→ [Positive] Schedule a call — draft a reply proposing times?
→ [Not now] Log and set reminder for 60 days — confirm?
→ [Unsubscribe] Mark OPT_OUT in CRM — confirm?
→ [Unclear] Read the full message and decide — what would you like to do?
```

---

### Step 3b: Unknown Sender — Notify

```
📬 Unknown sender reply — {email}

Subject: {subject}

--- Message ---
{body_text (truncated to 300 chars)}
---

This sender is not in CRM. Would you like to:
1. Add them as a new contact
2. Ignore
```

---

### Step 4: Mark Thread as Read

After notifying [Sales Rep], mark the thread as read to avoid duplicate alerts:

```bash
curl -s -X PATCH "https://api.openmail.sh/v1/threads/{thread_id}" \
  -H "Authorization: Bearer ${OPEN...Y}" \
  -H "Content-Type: application/json" \
  -d '{"is_read": true}'
```

---

### Step 5: Update CRM (after [Sales Rep] confirms)

**On [Sales Rep] confirmation**, update CRM:

**Positive interest / Not now / Unclear:**
```graphql
mutation {
  updateCompany(id: "{company_id}", data: {
    accountStatus: "WARM"
  }) { id }
}
```

**Unsubscribe:**
```graphql
mutation {
  updateCompany(id: "{company_id}", data: {
    accountStatus: "OPT_OUT"
  }) { id }
}
```

**Log engagement:**
```graphql
mutation {
  createEngagement(data: {
    name: "{YYYY-MM-DD} — Inbound Reply — {Company Name}"
    engagementType: "EMAIL"
    engagementStatus: "COMPLETED"
    engagementDate: "{received_date_iso}"
    companyId: "{company_id}"
    engagementNote: { json: ... }
  }) { id }
}
```

Link person as attendee:
```graphql
mutation {
  updateEngagement(id: "{engagement_id}", data: {
    clientAttendees: { connect: [{ id: "{person_id}" }] }
  }) { id }
}
```

Store `threadId` in `engagementNote` for future reply threading.

---

### Step 6: Suggested Follow-up Actions

Based on intent classification, proactively offer:

| Intent | Leo's offer |
|---|---|
| Positive interest | "Draft a reply proposing 3 meeting times?" |
| Not now | "Set a 60-day reminder task in CRM?" |
| Unsubscribe | "Mark OPT_OUT — no further outreach" |
| Unclear | "Would you like me to draft a clarifying reply?" |

Wait for [Sales Rep] confirmation before taking any action.

---

## Pitfalls

1. **Always mark thread as read** after notifying — prevents duplicate alerts on next cron run
2. **Never auto-reply** — Leo notifies, [Sales Rep] decides
3. **Unknown senders still get notified** — don't silently discard
4. **CRM update waits for human confirm** — never update status without explicit approval
5. **body_text may be empty** — if so, try `body_html` stripped of tags, or note "message body unavailable"
6. **Multiple messages in one thread** — read only the latest inbound message, not the full thread history
7. **Rate limit**: 1,000 API requests/min — no risk at this polling frequency
