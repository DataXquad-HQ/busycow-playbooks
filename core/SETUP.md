# Core Playbook — Setup

Run this after `setup/SETUP.md` is complete.

This setup installs the **three core pillars** of the BusyCow stack:
1. **GBrain** — long-term knowledge graph + Hermes system prompt refinement
2. **Lark MCP** — native Lark tool access for the agent
3. **Google Workspace** — external email, calendar, Drive (optional)

Then it creates the **Hermes Registry** base and installs all core skills.

---

## Step 1 — Initialize GBrain

GBrain is the long-term knowledge graph. All entities (people, companies, decisions, intel) live here.

### 1a. Install GBrain

Follow the GBrain installation guide:
```
https://github.com/nous-research/gbrain — follow README install steps
```

Confirm GBrain is running:
```bash
gbrain status
```

### 1b. Refine Hermes System Prompt

Append the GBrain routing rules to `~/.hermes/SOUL.md` so Hermes knows to read from and write to GBrain automatically. Add the following block:

```markdown
## GBrain Knowledge Routing

Before every response, silently check:
- Is a person, company, deal, or decision mentioned? → query GBrain first, write to GBrain after
- Is new intel being shared? → extract_facts to GBrain before responding
- Is a key decision being made? → put_page decisions/YYYY-MM-DD-topic to GBrain

Auto-write triggers (no user prompt needed):
- New contact or company → put_page people/ or companies/
- Opportunity or partnership stage change → add_timeline_entry
- Key decision reached → put_page decisions/
- Market or competitor intel → put_page + extract_facts

Query routing:
- Entity facts (who/what) → GBrain first
- Past workflow (how we did something) → session_search first
- Need both → GBrain then session_search
```

### 1c. Configure GBrain Source

Set the GBrain home directory in Hermes config:
```bash
hermes config set gbrain.home ~/.gbrain
```

Verify GBrain is accessible from Hermes:
```
Ask Hermes: "What's in my GBrain?"
→ Should return brain stats or empty brain confirmation
```

---

## Step 2 — Configure Lark MCP

This gives Hermes native Lark tool access (Bitable, Docs, Messaging) via MCP.

Fetch and follow the Lark MCP setup skill:
```
Fetch: https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/lark-mcp-setup.md
Install as skill, then follow its instructions.
```

After setup, verify:
```
Ask Hermes: "List my Lark chats"
→ Should return a list of groups/chats
```

---

## Step 3 — Google Workspace (optional)

Skip this step if you don't use Gmail / Google Calendar / Google Drive.

Fetch and follow the Google Workspace setup skill:
```
Fetch: https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/google-workspace.md
Install as skill, then follow its setup instructions to authenticate gws CLI.
```

After setup, verify:
```bash
gws gmail list
# Should return recent emails
```

---

## Step 4 — Create Hermes Registry Base in Lark

Call the Lark API to create a new Bitable app named **"Hermes Registry"**.

Inside this app, create two tables:

### Table: Skills

| Field | Type |
|-------|------|
| Name | Text (primary) |
| Description | Text |
| Source | Single Select: `our own` / `hermes` / `3rd-party` |
| Category | Single Select: `Sales` / `Internal Ops` / `Finance` / `System` / `Utility` / `Content` |
| Status | Single Select: `Active` / `Deprecated` |

### Table: Cron Jobs

| Field | Type |
|-------|------|
| Name | Text (primary) |
| Schedule | Text |
| Destination | Text |
| Status | Single Select: `Active` / `Paused` / `Disabled` |
| Last Run | DateTime |
| Notes | Text |

After creating, save to Memory:
```
Memory entry: "Hermes Registry Base: [app_token] | Skills: [table_id] | Cron Jobs: [table_id]"
```

---

## Step 5 — Install Core Skills

Fetch and install each skill by loading the raw URL and calling `skill_manage(action='create')`:

```
1. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/hermes-agent.md
2. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/managing-skills.md
3. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/managing-cron-jobs.md
4. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/maintaining-gbrain.md
5. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/maintaining-memory.md
6. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/capturing-to-gbrain.md
7. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/google-workspace.md
8. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/lark-mcp-setup.md
9. https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/core/skills/native-mcp.md
```

---

## Step 6 — Register Core Skills in Skills Registry

Add each skill to the Skills Registry Base:

| Name | Source | Category |
|------|--------|----------|
| hermes-agent | hermes | System |
| managing-skills | hermes | System |
| managing-cron-jobs | hermes | System |
| maintaining-gbrain | hermes | System |
| maintaining-memory | hermes | System |
| capturing-to-gbrain | hermes | System |
| google-workspace | hermes | System |
| lark-mcp-setup | hermes | System |
| native-mcp | hermes | System |

---

## Step 7 — Verify

- "What's in my GBrain?" → should return brain stats
- "List my installed skills" → should show 9 core skills
- "What's in the Skills Registry?" → should return 9 entries in Lark
- "List my Lark chats" → should return chat list (Lark MCP working)
- (If Google Workspace): `gws gmail list` → should return emails

---

## Next Step

Choose a business playbook:
- Sales & Ops: `https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/sales/SETUP.md`
- Internal Ops: `https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/internal-ops/SETUP.md`
- Knowledge Ops: `https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/knowledge-ops/SETUP.md`
