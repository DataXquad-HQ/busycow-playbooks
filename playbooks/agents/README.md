# Multi-Agent Operating System Playbook

This playbook sets up a full multi-agent team on top of the core Hermes + Lark + GBrain stack. Each agent is a separate Hermes profile with its own identity, memory, skills, and scope. Together they cover the full operating surface of a company: coordination, development, revenue, research, content, and technical support.

---

## Why Multiple Profiles

**Specialized memory.** Each agent carries only the knowledge it needs. A support agent remembers client server credentials and debugging patterns. A sales agent remembers deal stages and contact cadences. Keeping knowledge separate prevents contamination and keeps each agent's context dense with relevant signal.

**Parallel operation.** When multiple tasks need to run simultaneously — a research job, a content draft, a client debug session — separate profiles mean each task runs in its own session. A single agent handling all of this would be blocked for the duration of each task.

---

## Agent Profiles

| Profile | Role | Scope |
|---|---|---|
| `default` (Iris) | Chief of Staff | Coordination, dispatch, knowledge distillation, company direction |
| `steve` | Development Lead | Software development, CI/CD, infrastructure, autonomous coding |
| `leo` | Revenue | Sales pipeline, partnerships, lead enrichment, quotations |
| `quinn` | Researcher | Market intelligence, feasibility, competitive analysis, deep research |
| `maya` | Content & Growth | Blog, social media, pitch decks, GTM content, inbound funnel |
| `rex` | Support Engineer | Client technical support, remote debugging, incident response |

---

## What This Playbook Installs

1. All six Hermes profiles with SOUL.md and MEMORY.md
2. Shared skills installed via symlinks into each profile
3. Role-specific skills per profile
4. GBrain + Lark MCP configured for each profile
5. Cron jobs: Iris daily dispatcher, nightly dream cycle, task briefing, weekly audit
6. Lark Bitable schema: the full Structural Data model (Goals, Initiatives, Tasks, Accounts, Contacts, Activities, Opportunities, Partnerships, Quotations/Invoices)

---

## Files

- `SETUP.md` — agent-executable installation instructions
- `SCHEMA.md` — full Structural Data schema definition
- `profiles/` — SOUL.md and MEMORY.md templates per agent

→ Start with [`SETUP.md`](SETUP.md)
