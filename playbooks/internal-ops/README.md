# Internal Ops Playbook

The Internal Ops Playbook gives Hermes full task management capability — creating and tracking tasks, delivering daily briefings, reviewing priorities, and auditing task structure weekly.

## What's Included

| Skill | Trigger |
|-------|---------|
| `managing-tasks` | Create, update, or mark tasks done |
| `reviewing-tasks` | Task list, standup, workload overview by Goal |
| `planning-next-actions` | What should I focus on? Priority recommendation |
| `generating-task-briefing` | Daily briefing message (Mon–Fri, day-aware format) |
| `auditing-tasks` | Weekly Sunday structural health check |

## Architecture

```
Goals → Initiatives → Tasks
```
- **Goals** — 12–18 month business outcomes
- **Initiatives** — Focused workstreams toward a Goal
- **Tasks** — Concrete action items (only layer users need to input)

Hermes auto-assigns Initiative and Goal from task context — users never need to categorise manually.

## Prerequisites

- Core Playbook installed (`core/SETUP.md` complete)
- Lark connected with `bitable:app` and `im:message` permissions

## Install

```
Run this setup: https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/SETUP.md
```

## Example Interactions (after setup)

> "記下來：下週前要發提案給 [客戶]"
> "追蹤一下這件事：[description]"
> "今天有什麼任務？"
> "接下來最重要的事是什麼？"
> "幫我排一下今天"
> "Mark [task] as done"
