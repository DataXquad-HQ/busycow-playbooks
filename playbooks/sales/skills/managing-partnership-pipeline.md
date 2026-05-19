---
name: managing-partnership-pipeline
description: >
  Use when there is any update on a Partnership — new partner prospect, stage change, partner meeting, or contract trigger.
triggers:
  - "partnership update"
  - "partner stage"
  - "跟夥伴聊了"
  - "partner progress"
  - "有個新合作夥伴"
version: "2.0"
author: BusyCow
---

# Managing Partnership Pipeline

## What Is a Partnership?
A company with a formal/semi-formal relationship — reseller, tech partner, referral, channel.
Partner = the organisation. Individuals → Contacts table, linked via Accounts.

## Base
- **App Token:** `{{SALES_CRM_APP_TOKEN}}`
- **Partnership:** `{{PARTNERSHIP_TABLE_ID}}`
- **Activities:** `{{ACTIVITIES_TABLE_ID}}`

See `references/partnership-schema.md` and `references/activities-schema.md` for all field IDs.

## Stage Flow
Prospect → Qualifying → Agreement → Active / On Hold / Inactive

## Phase 1 — Triage
1. Identify every distinct partner mentioned
2. Classify: existing (update) or new (create)? Stage change? Concrete signal? Did partner bring an Opportunity?

Present parse back.

## Phase 2 — Probe Gaps
Before activity: Who, How, What, Partner response.
Before stage update: Concrete signal.
Before creating: Company name (in Accounts), Type, Country, Stage (default: Prospect).

## Phase 3 — Log Activity
Summary: "[Type] [Contact] @ [Partner] — [one-line outcome]"
Fields: Summary, Date (ms), Type, Account (DuplexLink), Client Response, Stage Advanced?, Next Action, Partnership (DuplexLink).

## Phase 4 — Update / Create Partnership
Existing: update Stage, append to Description narrative.
New: Check Accounts; generate ID PA-YYYY-NNN (query table for latest); set Name, Account, Stage, Description.

### Example Partners (replace with your actual partners)
- [Partner A] — reseller, e.g. regional distributor
- [Partner B] — tech partner, e.g. integration vendor

## Phase 5 — Push Next Steps
- 下一步？誰負責？→ managing-tasks
- Stage 對嗎？
- 需要起草合約？(Agreement stage)
- Partner 帶來商機？→ managing-sales-pipeline

## Pitfalls
- Never create duplicate — search by name before creating
- Activity Type must match exact SingleSelect
- DuplexLink = `{"link_record_ids": ["recXXX"]}`
- Date = millisecond timestamp (UTC+8)
- Always update Description when stage changes
