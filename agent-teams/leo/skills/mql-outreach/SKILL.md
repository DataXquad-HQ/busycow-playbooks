---
name: mql-outreach
version: 1.0
author: BD Lead Agent
description: >
  Draft and manage cold email outreach sequences for COLD Prospects in Twenty CRM.
  Segments prospects by entry source and drafts personalised sequences accordingly.
  Triggered when new prospects enter CRM (COLD status) or by daily cron checking
  for prospects due for next sequence step. Leo drafts, human sends.
  Use when: new COLD prospect needs outreach, or "start outreach for [company]",
  or daily outreach-sequence-check cron fires.
triggers:
  - start outreach
  - draft cold email
  - outreach sequence
  - 安排 outreach
  - 寫開發信
  - 冷郵件
---

## Purpose

For every COLD Prospect in CRM, draft a personalised cold email sequence matched to how they entered the pipeline. Track sequence progress. Update accountStatus to OUTREACH once first email is sent. Update to WARM when prospect responds.

Leo drafts every message. Human confirms before send. Leo never auto-sends.

---

## Sequence Design by Entry Source

Different entry paths warrant different tone and content:

| Entry source | Person.source value | Sequence approach |
|---|---|---|
| Cold list (LinkedIn, event exhibitor) | OUTBOUND_MAYA or EVENT | Cold intro sequence: introduce value prop, low-pressure CTA |
| Newsletter subscriber | OUTBOUND_MAYA | Content-led sequence: reference content they engaged with, move to conversation |
| Website enquiry / form fill | INBOUND_WEB | Fast warm response: acknowledge interest, ask about needs, propose call |
| Referral | REFERRAL | Warm intro sequence: reference who referred them, establish credibility |
| Human-introduced (event meeting) | EVENT or REFERRAL | Personal follow-up: reference where they met, specific talking point from conversation |

---

## Sequence Structure

Each sequence has up to 3 touches. Stop at first response.

| Touch | Timing | Purpose |
|---|---|---|
| Email 1 | Day 0 (immediately) | Introduce, establish relevance, one clear question |
| Email 2 | Day 4 (if no response) | Different angle, add value, softer CTA |
| Email 3 | Day 9 (if no response) | Final touch — direct ask or graceful close |

After Email 3 with no response: mark person as sequence-complete. Leave as OUTREACH status. Monthly lead-nurturing cron will re-engage after 30 days.

---

## Workflow

### Step 1: Identify Which Touch is Due

If triggered by cron (outreach-sequence-check):
```graphql
query {
  people(
    filter: {
      and: [
        { company: { accountStatus: { eq: "COLD" } } }
        { pointOfContactForOpportunities: { totalCount: { eq: 0 } } }
      ]
    }
    first: 50
  ) {
    edges {
      node {
        id
        name { firstName lastName }
        source
        jobTitle
        emails { primaryEmail }
        preferredChannel
        company { id name companyOverview accountStatus }
        engagementsAttended {
          edges {
            node { engagementDate engagementType }
          }
        }
      }
    }
  }
}
```

For each person:
- Count completed engagements with type EMAIL → this is their sequence position
- 0 emails sent → due for Email 1
- 1 email sent, last was 4+ days ago → due for Email 2
- 2 emails sent, last was 9+ days ago → due for Email 3
- 3 emails sent → sequence complete, skip

If triggered manually for a specific person: skip the scan, go straight to Step 2 for that person.

---

### Step 2: Pull Context for Draft

For each person due for outreach:
```graphql
query {
  person(id: "{person_id}") {
    id
    name { firstName lastName }
    jobTitle
    source
    emails { primaryEmail }
    preferredChannel
    remarks
    company {
      id name companyOverview enrichmentOverview
      country industry
    }
    engagementsAttended {
      edges {
        node {
          engagementDate engagementType
          outcome engagementNote { json }
        }
      }
    }
  }
}
```

---

### Step 3: Draft the Email

**Email 1 — Cold intro (for OUTBOUND_MAYA / EVENT source):**
```
Subject: {Company} — {one-line value relevance}

Hi {First Name},

[1 sentence: why reaching out — specific to their industry or role, not generic]

[1 sentence: what we do and who we help — concrete, not buzzword-heavy]

[1 genuine question relevant to their world — not a pitch]

[Light CTA — 15-min call / quick question / reply if relevant]

[Signature]
```
Target: under 100 words.

**Email 1 — Warm follow-up (for REFERRAL source):**
```
Subject: {Referrer name} suggested I reach out

Hi {First Name},

[1 sentence: {Referrer} mentioned you and why they thought we should connect]

[1 sentence: what we do, tied to the referrer's context]

[1 question or CTA]

[Signature]
```

**Email 1 — Fast response (for INBOUND_WEB source):**
```
Subject: Re: your enquiry about [topic]

Hi {First Name},

Thanks for reaching out. [1 sentence acknowledging what they asked about]

[1 question: what's the main thing you're trying to solve?]

[Propose a 15-min call this week]

[Signature]
```

**Email 2 — Different angle:**
```
Subject: {Company} — {different angle from Email 1}

Hi {First Name},

[Reference Email 1 briefly — "following up on my note last week"]

[New angle: a relevant insight, question, or use case — not a repeat of Email 1]

[CTA: even simpler — yes/no question]

[Signature]
```
Target: under 80 words.

**Email 3 — Final touch:**
```
Subject: {Company} — last note from me for now

Hi {First Name},

I'll keep this short — I've reached out a couple of times and haven't heard back. Totally fine if the timing isn't right.

If things change or you'd like to revisit, feel free to reach out.

[Signature]
```
Target: under 50 words.

---

### Step 4: Present Draft to Sales Rep

```
📬 Outreach Draft — {Company Name} / {Person Name}

Touch: Email {1/2/3} of 3
Source: {how they entered}
Last contact: {never / N days ago}

--- Draft ---
TO: {email}
SUBJECT: {subject}

{body}
---

Send this, edit it, or skip this contact?
```

For batch mode (cron), group all drafts and present together:
```
📬 Outreach Due Today — {N} contacts

1. {Name} ({Company}) — Touch {N} — {source}
   Subject: {subject preview}
   [Draft]

2. ...

Approve all / review one by one?
```

---

### Step 4a: Human Confirmation & Send via OpenMail

After presenting the draft, **wait for explicit human confirmation** before sending.

Accepted confirmations: "send", "yes", "ok", "確認發送", "寄出", or approving a specific contact by name.

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

On success response (`"status": "sent"`), capture:
- `messageId` — store for reference
- `threadId` — store on the CRM engagement record for future replies

**Report back to human:**
```
✅ Sent to {Name} <{email}>
messageId: {id}
threadId: {id}

Logging to CRM...
```

If API returns error: report the error, do NOT update CRM, ask human how to proceed.

---

### Step 5: After Send — Update CRM

After sales rep confirms send:

1. Log engagement:
```graphql
mutation {
  createEngagement(data: {
    name: "{YYYY-MM-DD} — Outreach Email {N} — {Company Name}"
    engagementType: "EMAIL"
    engagementStatus: "COMPLETED"
    engagementDate: "{send_date_iso}"
    companyId: "{company_id}"
  }) { id }
}
```

2. Link person as attendee:
```graphql
mutation {
  updateEngagement(id: "{engagement_id}", data: {
    clientAttendees: { connect: [{ id: "{person_id}" }] }
  }) { id }
}
```

3. If this is Email 1 (first ever outreach), update company accountStatus to OUTREACH:
```graphql
mutation {
  updateCompany(id: "{company_id}", data: {
    accountStatus: "OUTREACH"
  }) { id }
}
```

4. If prospect responds at any point, update to WARM:
```graphql
mutation {
  updateCompany(id: "{company_id}", data: {
    accountStatus: "WARM"
  }) { id }
}
```

---

### Step 6: Handle Response

If prospect responds:
1. Update accountStatus to WARM
2. Notify [Sales Rep] immediately
3. Suggest next step based on response tone:
   - Positive interest → propose Opportunity, ask [Sales Rep] to confirm
   - Wants more info → draft a follow-up with relevant context
   - Not now / wrong timing → log, set reminder for 60 days, leave as WARM
   - Unsubscribe / not interested → update to OPT_OUT

---

## Pitfalls

1. Always use localhost:3001 — never external URL
2. Leo drafts, human sends — never auto-send
3. Stop sequence immediately on any response — do not send Email 2 if Email 1 was answered
4. Sequence position is tracked by counting completed EMAIL engagements — not by a separate field
5. Email 3 is a graceful close, not a hard sell — tone matters
6. For INBOUND_WEB source, respond within same day if possible — high intent decays fast
7. Leave opportunity and partnership null on outreach engagements — these are not pipeline engagements yet
8. engagementNote is RICH_TEXT — use Twenty JSON format
