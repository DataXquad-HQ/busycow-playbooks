# BusyCow Playbooks

**Production-grade AI agent skills and setup guides for Hermes Agent.**

This repo packages everything needed to replicate a fully operational AI agent system — from initial stack setup to specialized business playbooks. Follow the installation order exactly. Each layer has its own SETUP.md that the agent reads and executes autonomously.

---

## Architecture

**Three core pillars:**

| Pillar | Purpose |
|---|---|
| **Lark / Feishu** | Workspace comms, databases (Bitable), documents |
| **Google Workspace** | External email, calendar, Drive, Sheets, Docs |
| **GBrain** | Long-term knowledge graph — entities, decisions, intel |

**Task management is business-anchored.** There is no standalone Task Tracker. Tasks live inside the business Base of the playbook that owns them (Sales & Ops, Finance, etc.), linked directly to Opportunities, Partnerships, or Initiatives. This keeps tasks tied to context, not floating in a separate system.

---

## Installation Flow

### Phase 0 — Human Prerequisites (manual, before running any agent)

These steps require a human. Complete all of them before running any automated setup.

**Step 1 — Provision a VM and install Hermes Agent**
```bash
# On your VM (Ubuntu 22.04+ recommended):
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Step 2 — Create a Lark organization and set up the Hermes Lark App**
1. Go to [open.feishu.cn](https://open.feishu.cn) (or Lark equivalent)
2. Create a new application (bot)
3. Set the App ID and App Secret in your Hermes config:
   ```bash
   hermes config set lark.app_id YOUR_APP_ID
   hermes config set lark.app_secret YOUR_APP_SECRET
   ```

**Step 3 — Enable Lark API permissions in the Dev Console**

In your Lark app's permission settings, enable:
- Messaging: `im:message`, `im:message:send_as_bot`
- Base (Bitable): `bitable:app`, `bitable:table`, `bitable:record`
- Docs: `docx:document`, `drive:drive`
- Contact: `contact:user.id:readonly`

Then publish/release the app to your organization.

**Step 4 — Run the Hermes Lark approval flow**

This links Hermes to your Lark user ID so it can send you messages:
```bash
hermes setup lark
```
Follow the prompts. At the end, Hermes will confirm your Lark user ID is in the approved list.

**Step 5 — (Optional) Google Workspace**

If you use Gmail / Google Calendar / Drive:
```bash
hermes setup google-workspace
```
Or set up manually via the `google-workspace` skill (installed in core).

---

### Phase 1 — Identity Setup (agent-automated)

```bash
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/setup/SETUP.md \
  | hermes "Follow these setup instructions exactly"
```

What this does:
- Writes `~/.hermes/SOUL.md` — knowledge routing rules (GBrain / Memory / Skills decision tree)
- Scaffolds `~/.hermes/MEMORY.md` — company name, Lark tokens, preferences
- Writes `~/.hermes/USER.md` — user profile

---

### Phase 2 — Core Setup (agent-automated)

```bash
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/SETUP.md \
  | hermes "Follow these setup instructions exactly"
```

What this does:
- Initializes **GBrain** knowledge graph and refines Hermes system prompt to use it
- Configures **Lark MCP** server for native tool access
- Sets up **Google Workspace** CLI (`gws`) if opted in
- Creates **Hermes Registry Base** in Lark (Skills table + Cron Jobs table)
- Installs all core skills

→ [`core/SETUP.md`](core/SETUP.md)

---

### Phase 3 — Business Playbooks (pick what you need)

Run each playbook SETUP.md to install the Lark Base schema, skills, chat groups, and cron jobs for that business context.

```bash
# Sales & Ops (CRM + pipeline + tasks)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/SETUP.md \
  | hermes "Follow these setup instructions exactly"

# Internal Ops (tasks, briefings, audits — requires Sales or a standalone Ops Base)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/SETUP.md \
  | hermes "Follow these setup instructions exactly"

# Knowledge Ops (GBrain enrichment, Lark → GBrain extraction, memory sync)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/knowledge-ops/SETUP.md \
  | hermes "Follow these setup instructions exactly"

# Finance (financial models, invoices, forecasts)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/finance/SETUP.md \
  | hermes "Follow these setup instructions exactly"

# Lark Ops (Bitable schema, Lark Docs, calendar sync)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/lark-ops/SETUP.md \
  | hermes "Follow these setup instructions exactly"

# Content (blog posts, pitch decks, diagrams, PDFs, multimedia)
curl -fsSL https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/content/SETUP.md \
  | hermes "Follow these setup instructions exactly"
```

---

## Core Skills

Foundation skills every workspace needs. Installed in Phase 2.

| Skill | What it does |
|---|---|
| `hermes-agent` | Complete guide to Hermes Agent — setup, config, spawning agents, troubleshooting |
| `managing-skills` | Create, update, delete skills and sync to Skills Registry |
| `managing-cron-jobs` | Schedule, update, and remove Hermes cron jobs |
| `maintaining-gbrain` | Run GBrain nightly dream maintenance cycle |
| `maintaining-memory` | Framework for deciding what goes in Memory vs Skills vs GBrain |
| `capturing-to-gbrain` | Save valuable conversation knowledge to GBrain |
| `google-workspace` | Gmail, Calendar, Drive, Sheets, Docs via `gws` CLI |
| `lark-mcp-setup` | Set up lark-mcp as a native MCP server in Hermes config |
| `native-mcp` | Connect MCP servers to Hermes — auto-discover and register tools |
| `managing-team-knowledge` | Log team decisions, track RACI ownership, detect Bus Factor risks |

---

## Business Playbooks

### Sales & Ops
CRM management, pipeline tracking, lead enrichment, quotation generation, partner management — and all business-linked tasks.

**Lark Base created:** `Sales & Ops` (Accounts, Contacts, Opportunities, Partnership, Activities, Goals, Initiatives, Tasks)
**Lark groups created:** `[Company] Sales`, `[Company] Ops`

| Skill | What it does |
|---|---|
| `capturing-sales-intel` | Add new contacts and companies to CRM |
| `enriching-leads` | Web-enrich new accounts with description, website, industry |
| `managing-sales-pipeline` | Create and update sales opportunities — stage, notes, follow-ups |
| `managing-partnership-pipeline` | Track partner prospects through agreement to active |
| `reviewing-sales-pipeline` | Pull pipeline status briefing — deals, invoices, forecast |
| `logging-sales-activities` | Log meeting notes and call summaries to CRM |
| `generating-quotations` | Generate quotation PDF from CRM data, upload to Drive |
| `managing-tasks` | Create and update tasks linked to Opportunities / Partnerships |
| `reviewing-tasks` | Query tasks with Goal-first prioritisation |

→ [`playbooks/sales/SETUP.md`](playbooks/sales/SETUP.md)

---

### Internal Ops
Task briefings, weekly audits, and prioritisation. **Requires Sales & Ops Base or a standalone Ops Base.**

**Lark groups created:** `[Company] Tasks`
**Cron jobs created:** Daily briefing (Mon–Fri 09:00), Weekly audit (Sun 09:00)

| Skill | What it does |
|---|---|
| `planning-next-actions` | Score active tasks and recommend top 3–5 actions with reasoning |
| `generating-task-briefing` | Generate daily task briefing message (Mon–Fri, format varies by day) |
| `auditing-tasks` | Weekly audit — orphan tasks, stale initiatives, grouping recommendations |

→ [`playbooks/internal-ops/SETUP.md`](playbooks/internal-ops/SETUP.md)

---

### Knowledge Ops
GBrain enrichment, knowledge extraction, team intelligence, and memory sync.

| Skill | What it does |
|---|---|
| `building-gbrain-knowledge-graph` | Enrich GBrain when brain score is low — entity pages, links, health check |
| `extracting-lark-to-gbrain` | Pull Lark group chat messages and extract knowledge into GBrain |
| `extracting-notion-pages` | Extract content from private Notion pages via API |
| `syncing-brain-memory` | Sync GBrain vault and Hermes memory files to GitHub |

→ [`playbooks/knowledge-ops/SETUP.md`](playbooks/knowledge-ops/SETUP.md)

---

### Lark Ops
Build and manage Lark/Feishu databases, documents, calendar sync, and financial tracking.

| Skill | What it does |
|---|---|
| `lark-bitable-schema-setup` | Create Bitable apps, tables, and fields via API |
| `lark-docx-writer` | Create structured Lark Docs programmatically |
| `reading-lark-files` | Download and read files shared via Lark links |
| `feishu-lark-bitable-calendar-sync` | Sync Bitable tasks to personal Lark Calendar |
| `tracking-financials` | Manage financial forecast and actuals in Lark Bitable |

→ [`playbooks/lark-ops/SETUP.md`](playbooks/lark-ops/SETUP.md)

---

### Content
Documents, presentations, diagrams, blog posts, PDF editing, and multimedia.

| Skill | What it does |
|---|---|
| `writing-blog-post` | Write structured blog posts from briefings |
| `marp-pitch-deck` | Build pitch decks using Marp markdown → PDF/PPTX |
| `excalidraw` | Create hand-drawn style diagrams in Excalidraw format |
| `powerpoint` | Create, edit, parse .pptx files |
| `ocr-and-documents` | Extract text from PDFs and scanned documents |
| `nano-pdf` | Edit PDFs with natural-language instructions |
| `youtube-content` | Fetch YouTube transcripts and transform into structured content |

→ [`playbooks/content/SETUP.md`](playbooks/content/SETUP.md)

---

### Finance
Financial modeling, investor forecasts, Google Sheets models, and invoice generation.

| Skill | What it does |
|---|---|
| `building-investor-financial-model` | Investor-grade financial forecast — multi-scenario Python model + Google Sheets |
| `gsheets-formula-model` | Live formula-driven Google Sheets model with scenario switcher |
| `generating-invoices` | Generate invoices from CRM data, fill Doc template, export PDF |

→ [`playbooks/finance/SETUP.md`](playbooks/finance/SETUP.md)

---

## Publisher

**BusyCow** — AI systems for operators who move fast.
https://github.com/DataXquad-HQ/busycow-playbooks
