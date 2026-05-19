---
name: enriching-leads
description: >
  Use when a new Account is added to the CRM, or when user asks to enrich an existing Account. Web-searches the company, finds description, website, industry, address, and confirms with user before writing to CRM.
triggers:
  - "enrich"
  - "幫我查一下這家公司"
  - "補一下客戶資料"
  - "lead enriching"
  - "查這間公司"
version: "1.0"
author: BusyCow
---

# Enriching Leads

## Accounts Table
- **App Token:** `{{SALES_CRM_APP_TOKEN}}`
- **Table ID:** `{{ACCOUNTS_TABLE_ID}}`

### Writable Fields
| Field | Field ID | Type |
|-------|----------|------|
| Client Name | `{{FIELD_CLIENT_NAME}}` | Text (primary) |
| Short Name | `{{FIELD_SHORT_NAME}}` | Text |
| Country | `{{FIELD_COUNTRY}}` | SingleSelect |
| Address | `{{FIELD_ADDRESS}}` | Text |
| Company Email | `{{FIELD_COMPANY_EMAIL}}` | Text |
| Description | `{{FIELD_DESCRIPTION}}` | Text |
| Industry | `{{FIELD_INDUSTRY}}` | SingleSelect |
| Website | `{{FIELD_WEBSITE}}` | Url |
| Company Size | `{{FIELD_COMPANY_SIZE}}` | SingleSelect |

### Industry Options
Configure to match your CRM's SingleSelect options. Example options:
科技/SaaS, 醫療/照護, 製造/代理, 水務/公用事業, 零售/電商, 物流/交通, 建築/地產, 金融/保險, 教育, 政府/公共機構, 餐飲/酒店, 其他

## Step 1 — Identify Account
Get company name. If record exists in CRM, fetch current fields first.

## Step 2 — Web Search
Run 1-2 searches: "[company_name] official website about", "[company_name] company profile industry"
Extract: website URL, description (2-3 sentences), industry, HQ address, company email, country.

## Step 3 — Confirm with User
Present findings before writing. Wait for confirmation.

## Step 4 — Write to CRM
Update Account record with confirmed fields.
Website field format: `{"link": "https://...", "text": "domain.com"}`

## Step 5 — Confirm to User
`✅ 已更新 [Company]: 網址、介紹、產業、地址 已寫入 CRM`

## Pitfalls
- Industry must exactly match one of the SingleSelect options configured in your CRM
- Website field is Url type — must pass `{"link": "...", "text": "..."}` not plain string
- Always confirm before writing — never auto-write without user approval
- If web search finds multiple companies with same name, ask user to clarify
