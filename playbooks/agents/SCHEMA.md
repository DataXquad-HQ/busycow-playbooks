# Structural Data Schema

This document defines the Structural Data model — the Lark Bitable tables that serve as the operational backbone for the multi-agent system. All agents read from and write to this shared schema.

---

## Design Principles

- **Customer-centred.** Every record traces back to an Account. Opportunities, Partnerships, Quotations, and Activities all link to Accounts and Contacts.
- **Task-anchored.** Tasks are the unit of work for both humans and agents. Every task links up to an Initiative, Opportunity, or Partnership — never floating.
- **Human vs Agent.** Tasks explicitly distinguish whether they are executed by a human or an agent, and which agent is assigned.
- **Single Base.** All tables live in one Lark Bitable Base. This enables cross-table DuplexLinks and keeps context unified.

---

## Layer 1: Strategy

### Goals
The top-level objectives. Everything else links up to a Goal.

| Field | Type | Notes |
|---|---|---|
| Goal Name | Text (index) | Short, outcome-oriented |
| Description | Text | What success looks like |
| Status | Single Select | Active / Achieved / Paused |
| Owner | Person | Human accountable |
| Target Date | Date | |
| Initiatives | DuplexLink → Initiatives | |

### Initiatives
Programmes of work that serve a Goal. Each Initiative contains multiple Tasks.

| Field | Type | Notes |
|---|---|---|
| Initiative Name | Text (index) | |
| Goal | DuplexLink → Goals | |
| Description | Text | Scope and expected output |
| Status | Single Select | Planning / Active / Done / Blocked |
| Owner | Person | |
| Due Date | Date | |
| Tasks | DuplexLink → Tasks | |

---

## Layer 2: Execution

### Tasks
The core unit of work. Shared across humans and agents.

| Field | Type | Notes |
|---|---|---|
| Task Name | Text (index) | |
| Worker Type | Single Select | **Human** / **Agent** |
| Assignee | Text | Person name or Agent name (Iris / Steve / Leo / Quinn / Maya / Rex) |
| Task Status | Single Select | To Do / In Progress / Blocked / In Review / Done |
| Priority | Single Select | Critical / High / Medium / Low |
| Due Date | Date | |
| Initiative | DuplexLink → Initiatives | |
| Opportunity | DuplexLink → Opportunities | |
| Partnership | DuplexLink → Partnerships | |
| Depends On | DuplexLink → Tasks (self) | For dependency tracking |
| Handoff Context | Long Text | Injected by dispatcher at assignment time — relevant GBrain + Lark context |
| Agent Notes | Long Text | Written by agent after completing the task |
| Result for Human | Long Text | Written by Iris/CoS after reviewing agent output — human-readable summary |
| Done | Checkbox | Set by the executing agent when complete |

---

## Layer 3: CRM

### Accounts
One record per company. Status field acts as the lead funnel stage.

| Field | Type | Notes |
|---|---|---|
| Account Name | Text (index) | |
| Status | Single Select | **Lead** / Prospect / Active / Churned / Blacklisted |
| Industry | Single Select | |
| Website | URL | |
| Description | Long Text | What the company does |
| Country / Region | Text | |
| Source | Single Select | Referral / Outbound / Inbound / Event / Other |
| Contacts | DuplexLink → Contacts | |
| Opportunities | DuplexLink → Opportunities | |
| Partnerships | DuplexLink → Partnerships | |
| Activities | DuplexLink → Activities | |
| Quotations | DuplexLink → Quotations & Invoices | |
| Tags | Multi Select | |
| Notes | Long Text | |

### Contacts
One record per person. Always linked to an Account.

| Field | Type | Notes |
|---|---|---|
| Contact Name | Text (index) | |
| Account | DuplexLink → Accounts | |
| Title / Role | Text | |
| Email | Email | |
| Phone | Phone | |
| LinkedIn | URL | |
| Decision Maker | Checkbox | Is this person a key decision maker? |
| Relationship Status | Single Select | Cold / Warm / Active / Inactive |
| Last Contacted | Date | |
| Notes | Long Text | |
| Activities | DuplexLink → Activities | |

### Activities
Log of every meaningful interaction — calls, emails, meetings, demos.

| Field | Type | Notes |
|---|---|---|
| Activity Title | Text (index) | Short description of what happened |
| Type | Single Select | Call / Email / Meeting / Demo / Event / Other |
| Date | Date | When it happened |
| Account | DuplexLink → Accounts | |
| Contact | DuplexLink → Contacts | |
| Opportunity | DuplexLink → Opportunities | |
| Partnership | DuplexLink → Partnerships | |
| Logged By | Text | Human or Agent name |
| Summary | Long Text | What was discussed, outcomes, next steps |
| Follow Up Required | Checkbox | |
| Follow Up Date | Date | |

---

## Layer 4: Deals

### Opportunities
Active or potential revenue deals.

| Field | Type | Notes |
|---|---|---|
| Opportunity Name | Text (index) | |
| Account | DuplexLink → Accounts | |
| Contact | DuplexLink → Contacts | |
| Stage | Single Select | Prospecting / Qualified / Proposal / Negotiation / Closed Won / Closed Lost |
| Value | Number | Estimated deal value |
| Currency | Single Select | USD / SGD / TWD / MYR / Other |
| Close Date | Date | Expected or actual close date |
| Probability | Number | 0–100% |
| Owner | Text | Human or Agent assigned |
| Tasks | DuplexLink → Tasks | |
| Activities | DuplexLink → Activities | |
| Quotations | DuplexLink → Quotations & Invoices | |
| Notes | Long Text | |

### Partnerships
Strategic partnerships — MoUs, reseller agreements, channel partners.

| Field | Type | Notes |
|---|---|---|
| Partnership Name | Text (index) | |
| Account | DuplexLink → Accounts | |
| Contact | DuplexLink → Contacts | |
| Status | Single Select | Exploring / MoU Signed / Active / Paused / Terminated |
| Type | Single Select | Reseller / Technology / Channel / Strategic / Other |
| Start Date | Date | |
| Review Date | Date | Next scheduled check-in |
| Owner | Text | Human or Agent assigned |
| Tasks | DuplexLink → Tasks | |
| Activities | DuplexLink → Activities | |
| Notes | Long Text | |

---

## Layer 5: Commercial Documents

### Quotations & Invoices
Track all commercial documents linked to Opportunities and Accounts.

| Field | Type | Notes |
|---|---|---|
| Document ID | Text (index) | e.g. QT-2026-001 or INV-2026-001 |
| Type | Single Select | **Quotation** / **Invoice** |
| Status | Single Select | Draft / Sent / Accepted / Rejected / Paid / Overdue / Cancelled |
| Account | DuplexLink → Accounts | |
| Opportunity | DuplexLink → Opportunities | |
| Issue Date | Date | |
| Due Date | Date | For invoices |
| Amount | Number | |
| Currency | Single Select | |
| Google Doc Link | URL | Link to the actual document in Google Drive |
| Notes | Long Text | |

---

## Summary Table

| Table | Purpose | Links to |
|---|---|---|
| Goals | Top-level objectives | Initiatives |
| Initiatives | Programmes of work | Goals, Tasks |
| Tasks | Unit of work (human or agent) | Initiatives, Opportunities, Partnerships |
| Accounts | Company records (incl. leads) | Contacts, Opportunities, Partnerships, Activities, Quotations |
| Contacts | Person records | Accounts, Activities |
| Activities | Interaction log | Accounts, Contacts, Opportunities, Partnerships |
| Opportunities | Revenue deals | Accounts, Contacts, Tasks, Activities, Quotations |
| Partnerships | Strategic agreements | Accounts, Contacts, Tasks, Activities |
| Quotations & Invoices | Commercial documents | Accounts, Opportunities |
