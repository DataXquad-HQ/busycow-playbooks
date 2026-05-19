---
name: auditing-tasks
description: >
  Weekly Sunday audit of the task structure — checks if tasks are under the right Initiative, finds orphan tasks, identifies stale Initiatives, and surfaces new Initiatives worth opening. Called by the Weekly Task Audit cron job every Sunday.
triggers:
  - "audit tasks"
  - "weekly task review"
  - "任務整理"
  - called by cron on Sunday
version: "1.0"
author: BusyCow
---

# Auditing Tasks

## Purpose
Not a status report — a structural health check.
Goal: ensure every task is in the right place, every Initiative is worth keeping, and nothing is slipping through the cracks.

## Base & Tables
- **App Token:** `{{TASK_TRACKER_APP_TOKEN}}`
- **Tasks:** `{{TASKS_TABLE_ID}}`
- **Initiatives:** `{{INITIATIVES_TABLE_ID}}`
- **Goals:** `{{GOALS_TABLE_ID}}`

## Delivery
Send audit report to the task group:
- `receive_id_type`: `chat_id`
- `receive_id`: `{{TASK_GROUP_CHAT_ID}}`
- `msg_type`: `text`

## Goals
Fetch all Goals from `{{GOALS_TABLE_ID}}` at runtime to get current record IDs and names.

## Phase 1 — Fetch All Data
1. Fetch all Tasks (Done = false)
2. Fetch all Initiatives (Status NOT IN Done)
3. Fetch all Goals
For each task: Is Initiative linked? Is Initiative's Goal linked? Business Line match?

## Phase 2 — Run 4 Checks

### Check A — Orphan Tasks (no Initiative)
Find tasks where Initiative field is empty.
Group by Business Line + keyword similarity.
For groups ≥3 with similar keywords → propose a new Initiative name.
For isolated orphans → fuzzy-match to existing Initiative; if no match → flag "需要手動分類".

### Check B — Misaligned Tasks
Find tasks where task's Business Line ≠ linked Initiative's Business Line.
→ Flag each one for review.

### Check C — Stale Initiatives
Find Initiatives where Status = Active BUT has 0 tasks, OR all tasks are Done/Cancelled, OR Target Finished date has passed.
→ Propose: mark as Done, On Hold, or archive.

### Check D — New Initiative Candidates
Look at orphan task clusters + recent task patterns.
Propose max 2–3 new Initiatives per run.

## Phase 3 — Generate Report

```
🔍 每週任務結構審查 — {date_str}

📊 概況
• 進行中任務：N 個 ｜ 無 Initiative：N 個 ｜ Initiatives：N 個

⚠️ 需要處理 (max 5 items)
• [Task] — 建議歸入「[Initiative]」

🗂 Initiative 健康 (max 3)
• 「[Initiative]」無進行中任務 → 建議標記 Done

💡 建議開啟 (max 2)
• 「[New Initiative]」— [N] 個相關任務，歸入 [Goal]

✅ 結構健康的 Initiatives（N 個，略）
---
如需調整，直接在這裡說，我幫你更新 ✍️
```

## Phase 4 — Auto-fix (safe changes only)
Auto-apply only: link orphan task to Initiative when confidence >90% AND only one plausible match; update Initiative Status to Done when ALL tasks are Done/Cancelled.
Everything else → surface in report, wait for human confirmation.

## Pitfalls
- DuplexLink field returns `{link_record_ids: [...]}` or null — check for null before accessing
- Lookup fields are read-only — can't write to them
- Don't create new Initiatives without surfacing them in the report first
