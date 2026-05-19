---
name: reviewing-sales-pipeline
description: >
  Pull live data from the Sales CRM Base and deliver a structured sales status briefing — pipeline health, deal stages, outstanding invoices, recent activity, and revenue forecast vs actuals. Use when user asks about sales status, pipeline overview, or wants to understand where the business stands commercially.
triggers:
  - "sales status"
  - "pipeline review"
  - "有什麼 deal"
  - "收款狀況"
  - "銷售概況"
  - "outstanding invoice"
  - "revenue 狀況"
version: "1.0"
author: BusyCow
---

# Reviewing Sales Pipeline

## Base & Tables
- **App Token:** `{{SALES_CRM_APP_TOKEN}}`
- **Opportunity:** `{{OPPORTUNITIES_TABLE_ID}}`
- **Activities:** `{{ACTIVITIES_TABLE_ID}}`
- **Invoices:** `{{INVOICES_TABLE_ID}}`
- **Contracts:** `{{CONTRACTS_TABLE_ID}}`
- **Clients:** `{{ACCOUNTS_TABLE_ID}}`
- **Revenue:** `{{REVENUE_TABLE_ID}}`

## Data Pull Sequence
Pull in parallel:
1. Opportunity (all records)
2. Activities (last 30 days, sort Date desc, page_size=20)
3. Invoices (all non-collected)
4. Contracts (active)

## Analysis Framework

### Section 1 — Hot Pipeline
Deals in Qualified / Proposal / Negotiation:
- List: Client | Description | Stage | Est. Value | Owner | Next Action
- Flag ⚠️ if no activity in last 14 days
- Flag 🔴 if next action overdue
- Summarise total pipeline value

### Section 2 — Deal Stage Summary
Count and group all Opportunities by Stage.
Comment on conversion rate.

### Section 3 — Outstanding Invoices (AR)
Invoices where Outstanding Balance > 0:
- Group by Client
- Show: Invoice ID | Amount | Due Date | Days Overdue
- Flag 🔴 overdue > 14 days, 🟡 due within 7 days
- Total outstanding AR

### Section 4 — Recent Activity (last 14 days)
List interactions from Activities table.
Note accounts with vs without recent touch.

### Section 5 — Revenue Snapshot
From Revenue table: this month forecast vs actual, running year total.

## Output Format
In user's language. Tables and bullets, not paragraphs.
Always end with top 3 actions that would most move the needle.

## Pitfalls
- Outstanding invoices: use Outstanding Balance field, not Invoice Amount
- Date fields are ms timestamps — convert before displaying
- If a deal has no Next Action → flag as incomplete data
- Activities Date field may be empty on some records — skip those for recency calculation
