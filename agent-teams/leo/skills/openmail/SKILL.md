---
name: openmail
description: "Send, receive, and reply to emails via OpenMail (openmail.sh) — agent-native email API. Use for cold outreach sequences, follow-ups, and reading inbound replies."
version: 1.0.0
author: leo
tags: [email, outreach, openmail, sales]
---

# OpenMail — Agent Email via openmail.sh

Leo's dedicated email inbox for all outbound sales communication.

## Credentials

- **API Key**: stored at `{{OPENMAIL_API_KEY_FILE}}`
- **Inbox ID**: `{{OPENMAIL_INBOX_ID}}`
- **From address**: `{{OPENMAIL_ADDRESS}}`
- **Base URL**: `https://api.openmail.sh/v1`

Load credentials before any API call:
```bash
OPENMAIL_API_KEY=$(cat {{OPENMAIL_API_KEY_FILE}})
OPENMAIL_INBOX_ID="{{OPENMAIL_INBOX_ID}}"
```

---

## 1. Send an Email

```bash
OPENMAIL_API_KEY=$(cat {{OPENMAIL_API_KEY_FILE}})
OPENMAIL_INBOX_ID="{{OPENMAIL_INBOX_ID}}"
IDEMPOTENCY_KEY=$(python3 -c "import uuid; print(uuid.uuid4())")

curl -s -X POST "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/send" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: ${IDEMPOTENCY_KEY}" \
  -d '{
    "to": "prospect@company.com",
    "subject": "Subject line here",
    "body": "<p>HTML body here.</p><p>Best,<br>Leo</p>"
  }'
```

**⚠️ IMPORTANT**: `Idempotency-Key` is REQUIRED on every send. Always generate a fresh UUID. Sending without it will fail or cause duplicates.

**Plain text alternative** (use `text` instead of `body`):
```bash
-d '{"to":"...","subject":"...","text":"Plain text body"}'
```

---

## 2. Reply in Thread

Use `threadId` to keep conversation in the same thread:

```bash
curl -s -X POST "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/send" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(python3 -c 'import uuid; print(uuid.uuid4())')" \
  -d '{
    "to": "prospect@company.com",
    "subject": "Re: Original Subject",
    "body": "<p>Follow-up content here.</p>",
    "threadId": "thr_XXXXXXXX"
  }'
```

---

## 3. Check Inbound Messages

List recent inbound messages (unread first):
```bash
curl -s "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/messages?direction=inbound&limit=20" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}"
```

List unread threads:
```bash
curl -s "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/threads?is_read=false" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}"
```

Read a full thread:
```bash
curl -s "https://api.openmail.sh/v1/threads/{thread_id}/messages" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}"
```

Mark thread as read:
```bash
curl -s -X PATCH "https://api.openmail.sh/v1/threads/{thread_id}" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"is_read": true}'
```

---

## 4. Check Outbound Sent

```bash
curl -s "https://api.openmail.sh/v1/inboxes/${OPENMAIL_INBOX_ID}/messages?direction=outbound&limit=50" \
  -H "Authorization: Bearer ${OPENMAIL_API_KEY}"
```

---

## 5. Integration with Sales Skills

### mql-outreach (Cold Sequence)
- Load this skill first
- Use **Send** (section 1) for Day 0 / Day 4 / Day 9 touches
- Store `thread_id` from first send response → use for follow-up touches
- STOP sequence if **Check Inbound** returns a reply from that sender

### follow-up-email
- Load this skill first
- Use **Reply in Thread** (section 2) if thread exists
- Use **Send** (section 1) for fresh follow-up with no prior thread

### outreach-sequence-check (cron)
- Load this skill first
- Use **Check Inbound** (section 3) to detect replies before sending next touch
- If reply detected → mark as read → log to CRM → skip sequence touch → update Opportunity status

---

## 6. Rate Limits

| Limit | Value |
|---|---|
| Send per minute | 10 |
| Send per day | 200 |
| API requests per minute | 1,000 |
| Monthly emails (free) | 3,000 |

---

## Pitfalls

- **Missing Idempotency-Key** → request may fail or duplicate. Always use fresh UUID.
- **`replyTo` custom header** → requires paid plan. Free plan will return `403 feature_requires_paid_plan`.
- **HTML body**: use `body` field. Plain text: use `text` field. Don't mix both.
- **Thread replies**: always include `threadId` to keep conversation grouped.
- **Rate limit**: max 10 sends/min — add `sleep 7` between sends in batch loops.
