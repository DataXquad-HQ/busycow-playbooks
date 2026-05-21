# Sales & Ops Playbook — Setup

Run this after `core/SETUP.md` is complete.

This setup:
1. Creates the **Sales & Ops Base** in Lark — one unified base for CRM + tasks
2. Installs sales and task management skills
3. Creates recommended Lark chat groups
4. Registers skills in the Skills Registry

**Design principle:** Tasks are business-anchored. They live here alongside Opportunities and Partnerships, not in a separate system.

---

## Step 1 — Create Sales & Ops Base

Call the Lark API to create a new Bitable app named **"Sales & Ops"**.

Then create the following tables:

### Table 1: Accounts
| Field | Type | Notes |
|-------|------|-------|
| Account Name | Text (primary) | Company full name |
| Short Name | Text | Internal alias |
| Country | Single Select | e.g. Taiwan / Hong Kong / Malaysia |
| Client Type | Multi Select | Direct / Reseller / Partner / End User |
| Industry | Text | |
| Website | URL | |
| Source | Single Select | How they came into pipeline |
| Business Line | Multi Select | Which products are relevant |
| Notes | Text | |

### Table 2: Contacts
| Field | Type | Notes |
|-------|------|-------|
| Contact Name | Text (primary) | |
| Company | DuplexLink → Accounts | |
| Role / Title | Text | |
| Decision Role | Single Select | Buyer / User / Influencer / Blocker / Champion |
| Phone | Phone | |
| Email | Email | |
| Preferred Channel | Single Select | WhatsApp / Email / Phone |
| LinkedIn | URL | |
| Notes | Text | |

### Table 3: Opportunities
| Field | Type | Notes |
|-------|------|-------|
| Order ID | Text (primary) | Format: OP-YYYY-NNN |
| Client | DuplexLink → Accounts | |
| Stage | Single Select | Lead / Qualified / Proposal / Negotiation / Closed Won / Closed Lost |
| Type | Single Select | New Business / Upsell / Renewal / Partnership |
| Business Line | Single Select | |
| Summary | Text | One-line description |
| Description | Text | Full notes and context |
| Expected Value | Number | |
| Expected Close Date | DateTime | |
| Next Action | Text | |
| Owner | User | |
| Tasks | DuplexLink → Tasks | |

### Table 4: Partnership
| Field | Type | Notes |
|-------|------|-------|
| ID | Text (primary) | Format: PA-YYYY-NNN |
| Name | Text | Partner company name |
| Stage | Single Select | Prospect / Qualifying / Agreement / Active / On Hold / Inactive |
| Type | Multi Select | Reseller / Distributor / Referral / Technology / OEM |
| Country | Single Select | |
| Summary | Text | One-line status |
| Description | Text | Running narrative log |
| Account | DuplexLink → Accounts | |
| Activities | DuplexLink → Activities | |
| Tasks | DuplexLink → Tasks | |

### Table 5: Activities
| Field | Type | Notes |
|-------|------|-------|
| Summary | Text (primary) | One-line description |
| Account | DuplexLink → Accounts | |
| Contact | Text | Person name |
| Type | Single Select | 電話 / 實體拜訪 / 線上會議 / WhatsApp\/LINE / Demo / 訊息 / Other |
| Client Response | Text | What they said / their reaction |
| Stage Advanced? | Checkbox | Did the deal move forward? |
| Next Action | Text | What needs to happen next |
| Date | DateTime | When it happened |
| Owner | User | Who conducted the interaction |
| Opportunity | DuplexLink → Opportunities | |
| Partnership | DuplexLink → Partnership | |

### Table 6: Goals
| Field | Type | Notes |
|-------|------|-------|
| Goal Name | Text (primary) | 12–18 month business outcome |
| Business Line | Single Select | Your business lines |
| Target Date | DateTime | Expected achievement date |
| Status | Single Select | Active / Achieved / On Hold |
| Notes | Text | |

### Table 7: Initiatives
| Field | Type | Notes |
|-------|------|-------|
| Initiative Name | Text (primary) | Focused workstream |
| Goal | DuplexLink → Goals | |
| Business Line | Single Select | |
| Status | Single Select | Active / Done / On Hold / Cancelled |
| Target Finished | DateTime | |
| Tasks | DuplexLink → Tasks | |

### Table 8: Tasks
| Field | Type | Notes |
|-------|------|-------|
| Task Name | Text (primary) | Format: [TAG] action description |
| Done | Checkbox | true = completed |
| Deadline | DateTime | |
| Business Line | Single Select | |
| Responsible Person | User | |
| Priority | Single Select | 🔴 High / 🟡 Medium / 🟢 Low |
| Description | Text | |
| Initiative | DuplexLink → Initiatives | |
| Opportunity | DuplexLink → Opportunities | |
| Partnership | DuplexLink → Partnership | |

After creating, save to Memory:
```
Memory entry: "Sales & Ops Base: [app_token]
  Accounts: [id] | Contacts: [id] | Opportunities: [id]
  Partnership: [id] | Activities: [id]
  Goals: [id] | Initiatives: [id] | Tasks: [id]"
```

---

## Step 2 — Create Initial Goals

Add at least one Goal per active business line. Each Goal should represent a 12–18 month business outcome.

Example: "Achieve [Business Line] $X ARR by [Date]"

---

## Step 3 — Create Lark Chat Groups

Create two chat groups:

| Group Name | Purpose |
|---|---|
| `[Company] Sales` | Sales updates, deal discussions, pipeline reviews |
| `[Company] Tasks` | Daily task briefings, weekly audits, action tracking |

Add Hermes bot to both groups. Save the chat IDs to Memory:
```
Memory entry: "Sales group chat_id: [chat_id]"
Memory entry: "Tasks group chat_id: [chat_id]"
```

---

## Step 4 — Install Sales & Task Skills

Fetch and install each skill:

```
1. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/capturing-sales-intel.md
2. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/managing-sales-pipeline.md
3. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/managing-partnership-pipeline.md
4. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/reviewing-sales-pipeline.md
5. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/enriching-leads.md
6. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/logging-sales-activities.md
7. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/skills/generating-quotations.md
8. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/managing-tasks.md
9. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/reviewing-tasks.md
```

---

## Step 5 — Register in Skills Registry

| Name | Source | Category |
|------|--------|----------|
| capturing-sales-intel | our own | Sales |
| managing-sales-pipeline | our own | Sales |
| managing-partnership-pipeline | our own | Sales |
| reviewing-sales-pipeline | our own | Sales |
| enriching-leads | our own | Sales |
| logging-sales-activities | our own | Sales |
| generating-quotations | our own | Sales |
| managing-tasks | our own | Internal Ops |
| reviewing-tasks | our own | Internal Ops |

---

## Step 6 — Verify

- "Show me my accounts" → should query Accounts table
- "Log a call with [name]" → should trigger `logging-sales-activities`
- "Add task: [description]" → should create task in Tasks table
- "What tasks do I have?" → should return open tasks grouped by Goal
- "What's our pipeline?" → should trigger `reviewing-sales-pipeline`

---

## Done

Your Sales & Ops system is ready. Example interactions:

> "剛跟 [公司] 的 [姓名] 開了個會，他們對我們很有興趣"
> "幫我記一個新客戶：[公司名]"
> "這個 partner 現在什麼狀況？"
> "新增一個 task：下週前要發提案給 [客戶]"
> "我們的 pipeline 現在怎樣？"
