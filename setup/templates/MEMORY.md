# MEMORY — BusyCow Agent Memory Template
# Instructions: Fill in the {{PLACEHOLDER}} values for your setup.
# This file is loaded into the agent's context every session.

→ Date rule: Whenever any date or time is relevant in conversation (invoices, scheduling, deadlines, logging, etc.), always verify the actual current Taiwan time via `TZ=Asia/Taipei date` before proceeding. Never assume or infer the date.
§
Skills Registry: {{SKILLS_REGISTRY_APP_TOKEN}} → {{TABLE_ID}}（Name/Description/Source/Category）。Skills 異動需同步此表。Skill 原則：(1) 只存未來會重複觸發的 SOP，一次性修復 → GBrain；(2) description 以 "Use when..." 為主；(3) 命名 gerund，工具名保留原名；(4) description 用 user 不用人名。
§
現有 active crons: e48c20e44c01 (GBrain Nightly Dream + Memory Sync, 每天 04:00 TWN), 6b9a820aba65 (Daily Task Briefing, 週一至週五 09:00 TWN → {{CHAT_ID}}), 0a29a00861cc (Weekly Task Audit, 每週日 09:00 TWN → {{CHAT_ID}}), 71bbd295c7ab (Nightly Lark → GBrain Extract, 每天 03:00 TWN → {{CHAT_ID}})。
§
## Tailscale: GCP busycow-hermes + PC DESKTOP-JF08KHM, {{TAILSCALE_USER}} tailnet. FireBridge: http://{{SERVER_IP}}:8088. the_owner NOPASSWD sudo on GCP.
§
[Product] GSheets (shared chun/morris/user@yourdomain.com): v18-Target={{GOOGLE_SHEET_ID}}; v18-base={{GOOGLE_SHEET_ID}}; v18Full={{GOOGLE_SHEET_ID}}.
## 檔案存放原則: 所有文件 → Google Drive（BusyCow Shared Drive）。Lark/Feishu → 只留 Base（多維表格），不在 Lark 建文件。
§
## Notion API
Token: {{NOTION_TOKEN}}
Use for all Notion page reads/writes. Header: Authorization: Bearer *** Notion-Version: 2022-06-28
§
Tasks now in Sales & Ops Base ({{LARK_APP_TOKEN}}):
Goals: `{{TABLE_ID}}` | Initiatives: `{{TABLE_ID}}` | Tasks: `{{TABLE_ID}}`
Tasks↔Opportunity (fldxTM0Op2) + Tasks↔Partnership (flderan4Kb) DuplexLinks added May 2026.
Old Task Tracker Base ({{LARK_APP_TOKEN}}) deleted.
ALWAYS write Tasks using field NAMES not IDs. DuplexLink = plain string array ["recXXX"].
§
Partnership table ({{TABLE_ID}}) — Stage field (field_id: fldu8TVcvX). Options: Prospect, Qualifying, Agreement, Active, On Hold, Inactive. ID format: PA-YYYY-NNN (next: PA-2026-009). Summary field (fldr5XCuaN), Description field (fldjMeFiw3).
Opportunity table ({{TABLE_ID}}) — ID format: OP-YYYY-NNN (next: OP-2026-014). Summary field (fldklERXej, 1-line), Description field (fldELp5uET, full notes).
§
## Data Routing Rules
WRITE: SOP/format/ID rule → Skill | Entity/person/company/decision/intel → GBrain | Env fact/tool pref needed every session → Memory | Task state/one-off fix → nowhere
QUERY: Entity facts (who/what/schema) → GBrain first | Past workflow/how-we-did-it → session_search first | Need both → GBrain then session_search
AUTO-WRITE to GBrain: new contact/client → put_page people/ or companies/ | opportunity/partnership stage change → add_timeline_entry | key decision → put_page decisions/YYYY-MM-DD-topic | new intel from conversation → extract_facts
§
Distify — Separate Singapore entity (not a BusyCow product line); global robot distribution platform connecting manufacturers to local Sales & Service Hubs. Currently managed together with BusyCow ops (shared CRM/tasks). Spin off when Hub partnerships develop. Current deals: MTR Hong Kong (OP-2026-012), Oman National Archives — disinfection robot (OP-2026-013).
§
## BusyCow Google Drive Structure (BusyCow Shared Drive: {{GOOGLE_DRIVE_ID}})
Naming conventions established May 2026:
- Root: `[DX]` prefix for company-level folders (floats above BL folders alphabetically)
- BL folders (BusyCow, [Product], etc.): plain names, no prefix
- Inside each BL: `00_Inbox`, `01_Core`, `02_Sales & Marketing`, `03_Commercial/Business`, `04_...`, `99_Archived`
- Projects: ALL in `[DX] Projects/` (not inside BL folders) — named `[BL] Project Name Year`
- Clients: ALL in `[DX] Clients/` — each client has BL subfolders inside (e.g. [Client]/BusyCow/)
- Hunter's rule: "find by client name first, not by BL first"