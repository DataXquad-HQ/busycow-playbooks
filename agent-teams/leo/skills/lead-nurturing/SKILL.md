---
name: lead-nurturing
description: >
  Identify people in Twenty CRM with no active Opportunity or Partnership who haven't
  been engaged in 30+ days. Draft personalised check-in messages. Triggered
  manually ("send a check-in to [person]") or by monthly cron (1st of month).
  Runs in Basic mode (no article) by default.
triggers:
  - "nurture"
  - "check-in"
  - "cold contacts"
  - "monthly nurture"
  - "send check-in"
version: "3.1"
author: Leo (BD Director Agent)
---

# Lead Nurturing

## Purpose

For every Person in Twenty CRM with no active Opportunity or Partnership and no engagement in 30+ days — draft a personalised check-in message for the sales rep to review and send.

NOTE: This skill is for re-engaging existing contacts who have gone quiet (30+ days). For new COLD Prospects entering the pipeline for the first time, use `mql-outreach` instead.

Not broadcast marketing. One-to-one personal outreach. **Leo never auto-sends.**

---

## CRM Reference

**Twenty CRM:** `http://localhost:3001` (always localhost)
**GraphQL endpoint:** `http://localhost:3001/graphql`

---

## Two Modes

**Mode A — Manual:** Sales rep says "send a check-in to [name]" → Leo drafts for that person.

**Mode B — Monthly scan (cron):** Scan all qualifying people, generate batch of drafts, deliver to sales rep for review.

---

## Detection Logic

A Person qualifies when ALL of these are true:
1. **No active Opportunity** — no linked opportunity with stage ≠ CLOSED_WON/CLOSED_LOST
2. **No active Partnership** — no linked partnership with stage = ACTIVE
3. **No engagement in 30+ days** — last engagementDate > 30 days ago or no engagement at all

---

## Workflow

### Step 1: Find People to Nurture

**Mode A:**
```graphql
query {
  people(filter: { name: { firstName: { like: "%{name}%" } } }) {
    edges {
      node {
        id
        name { firstName lastName }
        jobTitle
        emails { primaryEmail }
        preferredChannel
        remarks
        lastContactDate
        company { name companyOverview }
        pointOfContactForOpportunities {
          edges { node { id stage } }
        }
        primaryContactForPartnerships {
          edges { node { id stage } }
        }
        engagementsAttended {
          edges {
            node {
              engagementDate outcome
              engagementNote { json }
            }
          }
        }
      }
    }
  }
}
```

**Mode B — monthly scan:**
```graphql
query {
  people(
    filter: {
      lastContactDate: { lte: "{30_days_ago_iso}" }
    }
    first: 100
  ) {
    edges {
      node {
        id name { firstName lastName }
        preferredChannel lastContactDate
        remarks jobTitle
        company { name }
        pointOfContactForOpportunities { edges { node { stage } } }
        primaryContactForPartnerships { edges { node { stage } } }
      }
    }
  }
}
```

Then filter client-side:
- Skip if any linked opportunity has stage ≠ CLOSED_WON/CLOSED_LOST
- Skip if any linked partnership has stage = ACTIVE

---

### Step 2: Pull Context

For each qualifying person:
- Last engagement: date, type, outcome summary
- Remarks field (personal background, topics discussed)
- Company overview
- Preferred channel

---

### Step 3: Draft Message (Basic Mode)

**Email:**
```
Subject: [personalised — reflects check-in context, not "just checking in"]

Hi {Name},

[1–2 sentences: personalised greeting referencing last interaction or
 something relevant to their industry/role]

[1 genuine question relevant to their world]

[Light CTA — coffee chat, quick call, simple reconnect]

[Signature]
```
Target: under 120 words.

**WhatsApp / LINE:**
```
Hi {Name},

[1 sentence warm greeting]

[1 genuine question relevant to their industry or role]

[Light CTA]
```
3–4 lines max.

---

### Step 4: Deliver Drafts

**Mode A output:**
```
📬 Nurture Draft — {Person Name}

Channel: {Email / WhatsApp / LINE}
Last contact: {N days ago — brief summary}

--- Draft ---
{message body}
---

Should I send this, or would you prefer to send it yourself?
```

**Mode B output:**
```
📬 Monthly Nurture List — {N} people

1. {Name} ({Company}) — last contact: {N days ago}
   Channel: {channel}
   Draft: {first 40 chars...}

2. ...

Review one by one, or approve all for sending?
```

---

### Step 4a: Human Confirmation & Send via OpenMail

After presenting drafts, **wait for explicit human confirmation** before sending each message.

- Mode A (single draft): wait for "send", "yes", "ok", "確認", or "寄出"
- Mode B (batch): human can say "approve all" or name specific contacts. Do NOT bulk-send without explicit approval.

Do NOT send if human says "edit", "skip", "hold", or does not respond.

**Once confirmed, send via OpenMail API:**

```bash
OPENMAIL_API_KEY=$(cat ~/.hermes/profiles/leo/secrets/openmail_api_key)
OPENMAIL_INBOX_ID="0527f34e-65ad-4a02-adbc-e7872a9a921e"
IDEMPOTENCY_KEY=$(python3 -c "import uuid; print(uuid.uuid4())")

curl -s -X POST "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/send" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: ${IDEMPOTENCY_KEY}" \
  -d '{
    "to": "{recipient_email}",
    "subject": "{subject}",
    "body": "{html_body}"
  }'
```

On success (`"status": "sent"`), capture `messageId` and `threadId`.

**Report back:**
```
✅ Sent to {Name} <{email}>
messageId: {id}
threadId: {id}

Logging to CRM...
```

For batch: confirm each send one by one, report status per contact.

If API returns error: report the error, do NOT log to CRM, ask human how to proceed.

---

### Step 5: Log Engagement After Sending

After sending, create an Engagement record:

```graphql
mutation {
  createEngagement(input: {
    engagement: {
      name: "{YYYY-MM-DD} — Nurture — {Person Name}"
      engagementType: "EMAIL"
      engagementStatus: "COMPLETED"
      engagementDate: "{send_date_iso}"
      engagementNote: { json: ... }
      companyId: "{company_id}"
    }
  }) { id }
}
```

Link the person:
```graphql
mutation {
  updateEngagement(id: "{id}", input: {
    engagement: {
      clientAttendees: { connect: [{ id: "{person_id}" }] }
    }
  }) { id }
}
```

**Leave `opportunity` and `partnership` null** — nurture engagements must not appear in pipeline views.

Also update `lastContactDate` on the person:
```graphql
mutation {
  updatePerson(id: "{person_id}", input: {
    person: { lastContactDate: "{send_date_iso}" }
  }) { id }
}
```

---

## Pitfalls

1. **Always use localhost** — never external URL.

2. **Never auto-send** — always present drafts for human review.

3. **Leave opportunity/partnership null on nurture engagements** — intentional. Keeps nurture out of pipeline views.

4. **Basic mode is the default** — a genuine personalised check-in with no article is often more effective than a generic article share.

5. **Skip people with active opportunities** — those are managed by `deal-progressing`.

6. **`lastContactDate` null** — if null, treat as "never contacted" → always qualifies for nurture.

7. **`engagementNote` is RICH_TEXT** — use Twenty's JSON format for the field.
