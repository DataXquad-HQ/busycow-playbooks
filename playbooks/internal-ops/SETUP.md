# Internal Ops Playbook — Setup

Run this after a business playbook (Sales & Ops or equivalent) is installed.

This setup:
1. Installs task operations skills (planning, briefing, auditing)
2. Creates cron jobs for daily briefings and weekly audits
3. Registers skills in the Skills Registry

**Note:** Tasks live in the Sales & Ops Base (or your business Base of choice). This playbook does not create a separate Task Tracker — it adds the operational layer on top of existing business data.

---

## Prerequisite

Confirm the Tasks table and chat group are already set up:
- Tasks table exists in your Sales & Ops Base (or equivalent)
- Tasks group chat_id is saved in Memory

If not, run `playbooks/sales/SETUP.md` first.

---

## Step 1 — Install Internal Ops Skills

Fetch and install each skill:

```
1. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/planning-next-actions.md
2. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/generating-task-briefing.md
3. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/skills/auditing-tasks.md
```

---

## Step 2 — Configure Cron Jobs

Set up two recurring cron jobs using the `managing-cron-jobs` skill:

### 2a. Daily Task Briefing (Mon–Fri)
```
Schedule: weekdays at 09:00 (your timezone)
Prompt: Use the generating-task-briefing skill to send today's briefing to the task group.
Deliver to: [Tasks group chat_id from Memory]
```

### 2b. Weekly Task Audit (Sunday)
```
Schedule: every Sunday at 09:00 (your timezone)
Prompt: Use the auditing-tasks skill to run this week's task structure audit and send the report to the task group.
Deliver to: [Tasks group chat_id from Memory]
```

---

## Step 3 — Register in Skills Registry

| Name | Source | Category |
|------|--------|----------|
| planning-next-actions | our own | Internal Ops |
| generating-task-briefing | our own | Internal Ops |
| auditing-tasks | our own | Internal Ops |

---

## Step 4 — Verify

- "What should I focus on today?" → should return scored priority recommendation
- Check Hermes cron list → should show 2 new cron jobs

---

## Done

Example interactions:

> "接下來最重要的事是什麼？"
> "今天有什麼任務？"
> "Mark [task] as done"
