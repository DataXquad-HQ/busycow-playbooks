---
name: planning-next-actions
description: >
  Use when user asks what they should focus on next. Pulls active tasks, scores by Goal importance × urgency × priority, and outputs the top 3–5 actions with reasoning — not a list, but a clear recommendation.
triggers:
  - "接下來該做什麼"
  - "今天先做什麼"
  - "幫我排一下"
  - "what should I focus on"
  - "最重要的事"
version: "1.0"
author: BusyCow
---

# Planning Next Actions

## Purpose
Not a list — a recommendation. Tell the user what to do next and why.

## Base & Tables
- **App Token:** `{{TASK_TRACKER_APP_TOKEN}}`
- **Tasks:** `{{TASKS_TABLE_ID}}`
- **Initiatives:** `{{INITIATIVES_TABLE_ID}}`
- **Goals:** `{{GOALS_TABLE_ID}}`

---

## Scoring Logic

```
Score = Urgency × 3 + Priority × 2 + Goal_Weight × 1

Urgency: 5=overdue, 4=due today, 3=due this week, 2=due this month, 1=no deadline+High, 0=no deadline+Medium/Low
Priority: High=3, Medium=2, Low=1
Goal_Weight: Revenue-generating Goals=2, Internal/ops Goals=1
```

Sort descending by Score. Take top 5.

---

## Output Format

```
🎯 接下來最重要的 [N] 件事：

1. [Task Name] — [Owner]
   → 為什麼：[one-line reason]
   → 目標：[Goal > Initiative]

---
💡 [Optional: one insight about patterns — same owner, same blocker, same Goal]
```

Rules:
- Max 5 items
- Each item gets ONE reason sentence
- End with optional insight if there's a pattern
- If everything is low urgency → say so and ask if priorities need updating

---

## Pitfalls
- Don't recommend Blocked tasks as "do next"
- Don't recommend tasks where Responsible Person ≠ person asking (unless full team view requested)
