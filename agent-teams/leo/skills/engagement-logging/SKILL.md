---
name: engagement-logging
description: >
  Use when a human reports a customer or partner interaction. Accepts any input
  format — verbal description, pasted chat log, meeting notes, transcript, or
  document. Leo extracts summary, outcome, and next step, confirms with the human,
  then writes to Twenty CRM and GBrain. Use when user says "跟客戶聊了",
  "有個 meeting", "這是會議記錄", "貼一下對話", or provides any interaction content.
triggers:
  - "跟客戶聊了"
  - "有個 meeting"
  - "剛打了電話"
  - "partner 有動靜"
  - "log engagement"
  - "記錄互動"
  - "這是會議記錄"
  - "貼一下對話"
  - "這是對話記錄"
version: "4.1"
author: Leo (BD Director Agent)
---

# Engagement Logging

## Purpose

Capture every customer or partner interaction into Twenty CRM as an Engagement
record. Accepts any input format. Leo extracts the substance, confirms with the
human before writing, and ensures every engagement ends with a clear next step.

---

## CRM Reference

**Twenty CRM:** `http://localhost:3001` (always localhost)
**GraphQL endpoint:** `http://localhost:3001/graphql`

### Engagement Fields
| Field | Twenty field name | Type |
|---|---|---|
| Name (primary) | `name` | TEXT |
| Type | `engagementType` | SELECT |
| Status | `engagementStatus` | SELECT |
| Channel | `channel` | SELECT |
| Date | `engagementDate` | DATE_TIME |
| Notes | `engagementNote` | RICH_TEXT |
| Next Action | `nextAction` | TEXT |
| Outcome | `outcome` | TEXT |
| Company | `company` | RELATION → company |
| Opportunity | `opportunity` | RELATION → opportunity (optional) |
| Partnership | `partnership` | RELATION → partnership (optional) |
| Client Attendees | `clientAttendees` | RELATION → person |

### `engagementType` options
`PHONE` / `INPERSON` / `ONLINE` / `MESSAGING` / `DEMO` / `EMAIL` / `EVENT`

### `engagementStatus` options
`PLANNED` / `COMPLETED`

### `channel` options
`EMAIL` / `WHATSAPP` / `LINE` / `PHONE` / `IN_PERSON` / `ZOOM` / `TEAMS`

---

## Planned vs Completed

- **Planned** — future meeting already booked. Used by `meeting-prep` cron.
  `outcome` and `nextAction` can be left blank.
- **Completed** — interaction already happened. Triggers `deal-progressing`.

---

## Workflow

### Step 1: Accept Any Input Format

The human may provide:
- A verbal description ("跟 ABC 公司開了個會，聊了 proposal 的事")
- A pasted chat log (WhatsApp, Line, WeChat, email thread)
- Meeting minutes (text document)
- A meeting transcript
- A file or document attachment

**Do not ask the human to restructure the input.** Accept whatever they give and extract from it.

If the input is a file or attachment, read the full content first before proceeding.

---

### Step 2: Extract from Input

From the raw input, extract:

| Field | What to look for |
|---|---|
| **Company / Person** | Who was involved — company name, contact name(s) |
| **Date** | When did this happen — explicit date, or infer ("今天", "剛剛", "昨天") |
| **Type / Channel** | How they communicated — call, in-person, Zoom, WhatsApp, etc. |
| **Summary** | What was discussed — key topics, decisions, information exchanged |
| **Outcome** | What was the result — positive signal, objection, status change, agreement reached |
| **Next Step hints** | Anything mentioned as a follow-up action, deadline, or commitment |

Map company/person names to existing CRM records:
```graphql
query {
  companies(filter: { name: { like: "%{name}%" } }) {
    edges { node { id name } }
  }
}
```

---

### Step 3: Confirm Summary + Outcome with Human

**Do not write to CRM yet.** Present your extraction for confirmation:

```
📋 我的理解是這樣，確認一下：

**公司：** {company_name}
**聯絡人：** {person_name_if_known}
**時間：** {date}
**方式：** {type / channel}

**Summary：**
{2-3 sentence summary of what happened}

**Outcome：**
{what was the result — signal strength, agreement, objection, status}

這樣對嗎？有需要補充或修正的地方嗎？
```

Wait for the human's response. If they say OK, proceed. If they correct or add, update and re-confirm.

---

### Step 4: Confirm Next Step (nextAction) — MANDATORY

`nextAction` 是一句話的方向性摘要，不是任務清單。

格式：**"{Owner}: {action} by {deadline}"**
範例：
- "Hunter: send proposal to David by Friday"
- "客戶方 (Sarah): confirm internal budget by 2026-06-20"
- "Leo: prepare follow-up email today"

Summary/outcome 確認後，問：

```
還有一件事——這次之後的下一步方向是什麼？

我初步判斷是：{inferred_next_step_from_input_if_any}

誰負責、做什麼、什麼時候？
```

從輸入裡預填，請人類確認。一句話就好。

如果人類說「沒有下一步」或「先等等」——接受，在 engagement 裡記錄："No next action defined at this time."

**注意：** `nextAction` 只是方向性摘要。真正的工作項目（準備文件、研究、draft 等）由 Step 7 的 `task-management` 處理。

---

### Step 5: Write Engagement to Twenty CRM

Once summary, outcome, and next step are all confirmed:

```graphql
mutation {
  createEngagement(data: {
    name: "{YYYY-MM-DD} — {company_name} ({brief_context})"
    engagementType: "{type}"
    engagementStatus: "COMPLETED"
    channel: "{channel}"
    engagementDate: "{datetime_iso}"
    engagementNote: "{notes_as_richtext}"
    nextAction: "{owner}: {action} by {deadline}"
    outcome: "{outcome_summary}"
    companyId: "{company_id}"
    opportunityId: "{opportunity_id_or_null}"
    partnershipId: "{partnership_id_or_null}"
  }) {
    id
    name
  }
}
```

`nextAction` format: **"{Owner}: {what} by {when}"**
Examples:
- "Hunter: send proposal by Friday"
- "客戶方 (David): confirm budget by 2026-06-20"
- "Leo: draft follow-up email today"

Link client attendees if person records exist:
```graphql
mutation {
  updateEngagement(id: "{engagement_id}", data: {
    clientAttendees: { connect: [{ id: "{person_id}" }] }
  }) { id }
}
```

---

### Step 6: Mark Completed Tasks (if any)

If the human mentions completing an existing task during this interaction:

```graphql
query {
  tasks(filter: {
    and: [
      { taskTargets: { opportunity: { id: { eq: "{opportunity_id}" } } } }
      { status: { neq: "DONE" } }
    ]
  }) {
    edges { node { id title { text } } }
  }
}
```

Update each completed task to `DONE`.

---

### Step 7: Identify and Create Tasks → `task-management`

Engagement 寫入 CRM 後，呼叫 `task-management` skill，傳入：
- 完整 engagement 內容（summary、outcome、notes、confirmed nextAction）
- `opportunity_id` 或 `partnership_id`
- 當前 deal health（若已知）

`task-management` 負責：
1. 掃描 engagement 內容，識別**所有需要執行的工作項目**
2. 檢查是否有重複/未完成的 task 已存在
3. 建立 Task 記錄（title、due date、priority、agentAdvice）
4. 將 engagement 中提到已完成的任務標為 DONE
5. 回傳已建任務清單，供 Step 10 確認訊息使用

**`nextAction` ≠ Task**——nextAction 是方向性摘要（一句話），Task 是具體工作項目。兩者都要填，但分開處理。

常見例子：
- "需要準備報價" → Task: "Prepare quotation for {company} — {product}"
- "要研究他們的技術架構" → Task: "Research {company} tech stack before next meeting"
- "要跟 Kevin 確認定價" → Task: "Align with Kevin on pricing for {company}"
- "他們想看 demo" → Task: "Prepare demo for {company}"

---

### Step 8: Trigger deal-progressing

After engagement is saved, invoke `deal-progressing` with the opportunity ID.
Skip if this is a nurture engagement (no opportunity linked).

---

### Step 9: GBrain Sync

```python
# Add timeline entry
mcp_gbrain_add_timeline_entry(
    slug=f"companies/{company_slug}",
    date=engagement_date,
    summary=f"{engagement_type} — {outcome_summary}",
    detail=f"{full_notes}\n\nNext Action: {next_action}",
    source="twenty-crm"
)

# Extract facts from notes + outcome
mcp_gbrain_extract_facts(
    turn_text=f"{summary}\n{outcome}\n{next_action}",
    entity_hints=[f"companies/{company_slug}"]
)
```

---

### Step 10: Confirm to Human

```
✅ 已記錄：

**{company_name}** — {engagement_type} on {date}
Outcome: {outcome_summary}
Next Step: {nextAction}

Twenty CRM ✅ | GBrain ✅

{if tasks created:}
📋 已建立 {N} 個任務：
1. [{priority}] {title} — due {date}
   → {one-line agent advice}
2. [{priority}] {title} — due {date}
   → {one-line agent advice}
```

If zero tasks were identified, omit the task section entirely.

---

## Nurture Engagements (no Opportunity/Partnership)

When logging a check-in to a cold contact with no active deal:
- Set `company` and `clientAttendees` as normal
- Leave `opportunityId` and `partnershipId` null
- Still confirm summary, outcome, and next step
- Still create a task if next step is defined

---

## Verification Checklist

- [ ] Input read in full before extracting
- [ ] Summary and outcome confirmed by human
- [ ] Next step confirmed — owner, action, deadline all filled
- [ ] Engagement record created in Twenty CRM
- [ ] Client attendees linked (if person records exist)
- [ ] Completed tasks marked DONE
- [ ] Next-step task created with agentAdvice
- [ ] `deal-progressing` triggered (if opportunity linked)
- [ ] GBrain timeline entry + facts extracted
- [ ] Confirmation sent to human

---

## Pitfalls

1. **Never write to CRM before human confirms summary + outcome** — extract first, confirm second, write third.

2. **Never skip the next step question** — even if the input mentions a next action, explicitly confirm it with owner + deadline. "Follow up" is not a next step. "Hunter sends proposal by Friday" is.

3. **`nextAction` field format** — always "{Owner}: {action} by {deadline}". This format makes the task scannable in the pipeline view.

4. **`engagementNote` is RICH_TEXT** — Twenty stores as JSON. Wrap plain text:
   `{ "root": { "children": [{ "children": [{ "text": "...", "type": "text" }], "type": "paragraph" }], "type": "root" } }`

5. **Always use localhost** — never external URL.

6. **Don't fabricate outcome signals** — if the input is ambiguous about how the interaction went, note it as "Outcome unclear from available notes" and ask the human.

7. **Long inputs (transcripts, documents)** — read fully before extracting. Do not truncate. Key signals often appear at the end of a long conversation.

8. **`engagementDate` format** — ISO 8601: `"2026-06-12T14:30:00.000Z"`.
