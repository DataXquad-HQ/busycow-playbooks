# Internal Ops Playbook — Setup

Run this after `core/SETUP.md` is complete.

This setup:
1. Creates the **Task Tracker Base** in Lark with 3 tables
2. Installs 5 internal ops skills
3. Sets up daily and weekly cron jobs
4. Registers skills in the Skills Registry

---

## Step 1 — Create Task Tracker Base

Call the Lark API to create a new Bitable app named "Task Tracker":

```
Create Lark Bitable app: "Task Tracker"
```

Then create the following tables inside this app:

### Table 1: Goals
| Field | Type | Notes |
|-------|------|-------|
| Goal Name | Text (primary) | 12–18 month business outcome |
| Business Line | Single Select | Your business lines |
| Target Date | DateTime | Expected achievement date |
| Status | Single Select | Active / Achieved / On Hold |
| Notes | Text | |

### Table 2: Initiatives
| Field | Type | Notes |
|-------|------|-------|
| Initiative Name | Text (primary) | Focused workstream |
| Goal | DuplexLink → Goals | |
| Business Line | Single Select | |
| Status | Single Select | Active / Done / On Hold / Cancelled |
| Target Finished | DateTime | |
| Tasks | DuplexLink → Tasks | |

### Table 3: Tasks
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
| Goal (lookup) | Lookup → Initiative → Goal | Read-only computed |

After creating, save to Memory:
```
Memory entry: "Task Tracker Base: [app_token] | Goals: [id] | Initiatives: [id] | Tasks: [id]"
```

---

## Step 2 — Create Initial Goals

Add at least one Goal per active business line. Each Goal should represent a 12–18 month business outcome.

Example: "Achieve [Business Line] $X ARR by [Date]"

Save the record IDs for each Goal — they are needed for Initiative assignment.

---

## Step 3 — Install Internal Ops Skills

Fetch and install each skill:

```
1. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/managing-tasks.md
2. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/reviewing-tasks.md
3. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/planning-next-actions.md
4. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/generating-task-briefing.md
5. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/auditing-tasks.md
```

---

## Step 4 — Configure Cron Jobs

Set up two recurring cron jobs:

### 4a. Daily Task Briefing (Mon–Fri)
```
Schedule: weekdays at 09:00 (your timezone)
Prompt: Use the generating-task-briefing skill to send today's briefing to the task group.
Deliver to: [your task group chat_id]
```

Save the chat_id to Memory:
```
Memory entry: "Task group chat_id: {{TASK_GROUP_CHAT_ID}}"
```

### 4b. Weekly Task Audit (Sunday)
```
Schedule: every Sunday at 09:00 (your timezone)
Prompt: Use the auditing-tasks skill to run this week's task structure audit and send the report to the task group.
Deliver to: [your task group chat_id]
```

Use `managing-cron-jobs` skill to create these, or set them up manually via Hermes cron commands.

---

## Step 5 — Register in Skills Registry

Add each skill to the Skills Registry (from Core Playbook):

| Name | Source | Category |
|------|--------|----------|
| managing-tasks | our own | Internal Ops |
| reviewing-tasks | our own | Internal Ops |
| planning-next-actions | our own | Internal Ops |
| generating-task-briefing | our own | Internal Ops |
| auditing-tasks | our own | Internal Ops |

---

## Step 6 — Verify

- "Add a task: [description]" → should create a task in Lark
- "What tasks do I have?" → should return current open tasks grouped by Goal
- "What should I focus on?" → should return scored priority recommendation

---

## Done

Your Internal Ops system is ready. Example interactions:

> "記下來：下週前要發提案給 [客戶]"
> "追蹤一下：需要跟 [partner] 確認合約進度"
> "今天有什麼任務？"
> "接下來最重要的事是什麼？"
> "Mark [task] as done"
