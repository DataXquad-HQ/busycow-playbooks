---
name: generating-task-briefing
description: >
  Generate the daily task briefing message for the task group. Determines today's day of week and outputs the appropriate format — Monday includes weekly highlights, Wednesday is mid-week check-in, Friday invites reflections. Called by the Daily Task Briefing cron job.
triggers:
  - called by cron
  - "daily briefing"
  - "發今天的 briefing"
version: "1.0"
author: BusyCow
---

# Generating Task Briefing

## Base & Tables
- **App Token:** `{{TASK_TRACKER_APP_TOKEN}}`
- **Tasks:** `{{TASKS_TABLE_ID}}`
- **Initiatives:** `{{INITIATIVES_TABLE_ID}}`
- **Goals:** `{{GOALS_TABLE_ID}}`

## Delivery
Send briefing to the task group:
- `receive_id_type`: `chat_id`
- `receive_id`: `{{TASK_GROUP_CHAT_ID}}`
- `msg_type`: `text`

## Step 1 — Determine Day of Week
```python
from datetime import datetime, timezone, timedelta
tz = timezone(timedelta(hours=8))
day = datetime.now(tz).strftime('%A')
date_str = datetime.now(tz).strftime('%-m/%-d')
```

## Step 2 — Fetch Tasks
Fetch all tasks, filter Done = false.
Classify into: overdue, due_today, high_no_date, this_week, rest.

## Step 3 — Generate Message by Day

### Monday — Week Start
```
🗓 {date_str} 週一｜本週開始
🔴 今天到期 / 逾期 ... 
⚡ 高優先、未排時間 ...
📋 本週重點 ...
---
📝 新的一週，請大家更新一下任務進展 ✍️
```

### Tuesday / Thursday — Standard
```
🗓 {date_str} {weekday_zh}
🔴 今天到期 / 逾期 ...
⚡ 高優先、未排時間 ...
📋 本週進行中 ...
```

### Wednesday — Mid-week
```
🗓 {date_str} 週三｜週中盤點
🔴 今天到期 / 逾期 ...
⚡ 高優先、未排時間 ...
---
📝 週中了，有沒有卡住的？直接說，我幫你記 ✍️
```

### Friday — End of Week
```
🗓 {date_str} 週五｜週末前
🔴 今天到期 / 逾期 ...
⚡ 高優先、未排時間 ...
---
📝 週末前，請大家更新任務進度 ✍️
```

## Formatting Rules
- Max 15 lines (excluding nudge section)
- Each task: `• [Task Name] — [Owner]（[Deadline or 無期限]）`
- Empty section → write `（無）`, don't skip the header
- Nudge section at bottom, separated by `---`

## Pitfalls
- Use `timezone(timedelta(hours=8))` from stdlib — do NOT use pytz (may not be installed)
- Deadline field returns ms timestamp — divide by 1000 for Python datetime
- Responsible Person field: get display name from `[0]['en_name']` — shorten to first name
- If no tasks at all → still send message with all sections showing `（無）`
- Large responses (150+ records) can cause issues — write complex Python to `/tmp/script.py` and execute separately
- Always fetch with page_size=100 and check `has_more` — paginate if true
