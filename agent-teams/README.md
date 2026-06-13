# Agent Teams

Per-agent configuration, capabilities, and skills.

Each subdirectory is a **self-contained agent package** — everything needed
to install and run one agent on a Hermes profile.

---

## Agent Roster

| Agent | Directory | Role |
|-------|-----------|------|
| Iris | `iris/` | Chief of Staff — coordination, dispatch, knowledge distillation |
| Leo | `leo/` | Revenue — sales pipeline, partnerships, lead enrichment |
| Maya | `maya/` | Content & Growth — blog, social, GTM |
| Rex | `rex/` | Customer Success — support, renewals, response |
| Steve | `steve/` | Development Lead — software, infrastructure |

---

## Directory structure per agent

```
agent-teams/<agent>/
├── SOUL.md          ← Role definition, identity, escalation rules
├── CAPABILITY.md    ← What this agent does, how to trigger it
├── CONTEXT.md       ← Data sources, GBrain config, operational context
├── SETUP.md         ← Step-by-step profile setup (agent-executable)
└── skills/          ← Agent-specific Hermes skills
    └── *.md
```

---

## Reading order (when installing an agent)

1. `CAPABILITY.md` — understand what the agent does and its authority
2. `CONTEXT.md` — prepare the data sources and docs it needs
3. `SOUL.md` — write to `~/.hermes/profiles/<agent>/SOUL.md`
4. `SETUP.md` — run the install steps

---

## Shared skills

Agents also use skills from `../shared-skills/` — install those first.
Agent-specific skills in `skills/` are additive on top of the shared layer.
