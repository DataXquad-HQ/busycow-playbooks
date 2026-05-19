---
name: reviewing-tasks
description: >
  Query and summarise tasks from the Task Tracker with Goal-first prioritisation. Use when user asks "what tasks do I have", "what's due today", "standup", "任務清單", "本週重點", or wants a workload overview grouped by Goal.
triggers:
  - "what tasks"
  - "what's due"
  - "standup"
  - "任務清單"
  - "today's tasks"
  - "task overview"
  - "本週重點"
version: "2.0"
author: BusyCow
---

# Reviewing Tasks

## Base & Tables
- **App Token:** `{{TASK_TRACKER_APP_TOKEN}}`
- **Tasks:** `{{TASKS_TABLE_ID}}`
- **Initiatives:** `{{INITIATIVES_TABLE_ID}}`
- **Goals:** `{{GOALS_TABLE_ID}}`

---

## Query Pattern

1. Fetch all tasks (page_size=100, paginate if has_more)
2. Filter: Done = false
3. For each task, resolve Initiative name + Goal name via linked record IDs
4. Apply Goal-first grouping and sort

---

## Goal-First Output Structure

```
🔴 今天到期 / 已逾期
  • [Task] — [Owner] — [Goal > Initiative]

⚡ 高優先、未排時間
  • [Task] — [Owner] — [Goal > Initiative]

📋 按 Goal 分組
  [Goal Name]
    [Initiative Name]
      • [Task] — [Owner] — [Deadline / no deadline]
```

Rules:
- Overdue = deadline < TODAY_START → surface first, mark ⚠️
- Due today = TODAY_START ≤ deadline ≤ TODAY_END → 🔴
- High priority + no deadline → ⚡
- Everything else → grouped by Goal → Initiative
- Within each group: sort by deadline asc, then priority

---

## Date Windows (UTC+8)

```python
from datetime import datetime, timezone, timedelta
tz = timezone(timedelta(hours=8))
today = datetime.now(tz).date()
TODAY_START = int(datetime(today.year, today.month, today.day, 0, 0, 0, tzinfo=tz).timestamp() * 1000)
TODAY_END   = int(datetime(today.year, today.month, today.day, 23, 59, 59, tzinfo=tz).timestamp() * 1000)
```

---

## Pitfalls
- Lookup fields (Goal on Tasks table) are read-only — don't try to write them
- Initiative DuplexLink returns array of record objects — resolve name via separate fetch if needed
- Empty deadline = no deadline, not overdue
