---
name: managing-tasks
description: >
  Create and update tasks in the [Company] Task Tracker (Lark Base). Triages
  multi-task dumps automatically — extracts every action item from a message,
  auto-assigns Initiative and Goal, and saves in one batch. Use when user says
  "記下來", "追蹤一下", "寫進去", "add a task", "update task", "mark done",
  or mentions anything that needs to be tracked as an action item.
triggers:
  - "記下來"
  - "追蹤一下"
  - "寫進去"
  - "add a task"
  - "update task"
  - "mark done"
  - "task status"
  - "new task"
version: "2.0"
author: BusyCow
---

# Managing Tasks

## Base & Tables
- **App Token:** `{{TASK_TRACKER_APP_TOKEN}}`
- **Tasks:** `{{TASKS_TABLE_ID}}`
- **Initiatives:** `{{INITIATIVES_TABLE_ID}}`
- **Goals:** `{{GOALS_TABLE_ID}}`

## Architecture
```
Goals → Initiatives → Tasks
```
- **Goal** — 12–18 month business outcome. Rarely changes.
- **Initiative** — Focused workstream toward a Goal.
- **Task** — Concrete action item. The only layer users need to input.

Hermes auto-assigns Initiative and Goal. Never ask user to assign manually.

---

## Phase 1 — Triage

Read the full message. Extract **every distinct action item** mentioned.
A single message may contain 3–5 tasks across different people and Business Lines.

For each extracted task, identify:
- What needs to be done (task name)
- Who is responsible (default: the person speaking)
- Any deadline or urgency signal
- Business Line + keyword signals for Initiative matching

Present parse back silently — do NOT ask for confirmation unless genuinely ambiguous.
Save all tasks, then confirm in one summary at the end.

---

## Phase 2 — Initiative & Goal Auto-Assignment

Every task **must** be linked to an Initiative and Goal. Never leave blank.

### Match Logic
| Confidence | Action |
|------------|--------|
| >80% | Link silently, mention inline: "→ 歸入 [Initiative]" |
| Ambiguous | Ask once: "歸入 [X] 還是 [Y]？" |
| No match | Propose new Initiative, wait for confirmation, then create |

### Goal Record IDs
Fetch all Goals from `{{GOALS_TABLE_ID}}` at runtime to get current record IDs.

---

## Phase 3 — Save to Lark

### Tasks Table Fields
| Field | Field ID | Type | Notes |
|-------|----------|------|-------|
| Task Name | {{FIELD_TASK_NAME}} | Text (primary) | Format: `[TAG] action description` |
| Done | {{FIELD_DONE}} | Checkbox | `true` = completed |
| Deadline | {{FIELD_DEADLINE}} | DateTime | ms timestamp UTC+8 |
| Business Line | {{FIELD_BUSINESS_LINE}} | SingleSelect | your business lines |
| Responsible Person | {{FIELD_RESPONSIBLE}} | User | `[{"id": "open_id"}]` + user_id_type: open_id |
| Priority | {{FIELD_PRIORITY}} | SingleSelect | High / Medium / Low |
| Description | {{FIELD_DESCRIPTION}} | Text | |
| Initiative | {{FIELD_INITIATIVE_LINK}} | DuplexLink → Initiatives | `{"link_record_ids": ["recXXX"]}` |

### Save order (if new Initiative needed)
1. Create Initiative in `{{INITIATIVES_TABLE_ID}}` → get record_id
2. Create Task(s) linked to new Initiative

---

## Phase 4 — Confirm

After saving all tasks, output one summary table:
```
✅ 已記錄 N 個任務：
| Task | Initiative | Responsible | Deadline | Priority |
```

---

## Updating Tasks
- Search by Task Name if no record ID
- To mark done: set Done = true
- After marking Done → ask if there's a next action

## Pitfalls
- User field MUST be array: `[{"id": "..."}]` + param `user_id_type: open_id`
- DuplexLink = `{"link_record_ids": ["recXXX"]}`
- Deadline = ms timestamp UTC+8
- Default Priority = Medium if not specified
- Default Done = false for new tasks

## Timestamp (UTC+8)
```python
from datetime import datetime, timezone, timedelta
tz = timezone(timedelta(hours=8))
ms = int(datetime(year, month, day, tzinfo=tz).timestamp() * 1000)
```
