---
name: managing-sales-pipeline
description: >
  Use when there is any update on a Sales Opportunity — new deal, stage change,
  client meeting, follow-up, or document trigger (quote/invoice). Handles both
  creating new Opportunities and updating existing ones.
triggers:
  - "跟客戶聊了"
  - "有進展"
  - "update stage"
  - "deal update"
  - "有個新機會"
  - "opportunity update"
  - "推進"
  - "pipeline update"
version: "2.0"
author: BusyCow
---

# Managing Sales Pipeline

## Base
- **App Token:** `{{SALES_CRM_APP_TOKEN}}`
- **Opportunity:** `{{OPPORTUNITIES_TABLE_ID}}`
- **Activities:** `{{ACTIVITIES_TABLE_ID}}`

See `references/opportunity-schema.md` and `references/activities-schema.md` for all field IDs.

## Stage Flow
Lead → Qualified → Proposal → Negotiation → Won / Lost

## Phase 1 — Triage the Update
1. Read the full message carefully
2. Identify every distinct company/deal mentioned
3. For each entity, classify: Existing opportunity (update) or new (create)? Stage change? Client signal? Next action?

Present parse back: "我看到 3 個更新：[A] — [B] — [C]。有沒有遺漏？"

## Phase 2 — Probe Gaps
Must have before logging activity: Who, How, What happened, Client response.
Must have before stage update: Concrete signal + new stage.
Must have before creating new Opportunity: Client (in Accounts), Description, Business Line, Stage (default: Lead), Owner.

## Phase 3 — Log Activity
Summary format: "[Type] [Contact] @ [Company] — [one-line outcome]"
Activity fields: Summary, Date (ms timestamp), Type, Account (DuplexLink), Client Response, Stage Advanced?, Next Action, Opportunity (DuplexLink).

## Phase 4 — Update / Create Opportunity
Existing: update Stage, Expected Value, Notes where new info available.
New: Check Accounts table; generate ID OP-YYYY-NNN (query table for latest); set Description, Client, Stage, Business Line, Owner.

## Phase 5 — Push Next Steps
After logging, ALWAYS push for next actions:
- 下一步？誰負責，什麼時候？→ managing-tasks
- Stage 對嗎？
- Document trigger: Proposal stage → generating-quotations; Won/payment → generating-invoices

## Pitfalls
- Activity Type must match exact SingleSelect option
- Date = millisecond timestamp (UTC+8)
- DuplexLink = `{"link_record_ids": ["recXXX"]}`
- Owner field is User type: `[{"id": "{{USER_OPEN_ID_OWNER}}"}]` + user_id_type: open_id
- Never create Opportunity without a linked Client record
