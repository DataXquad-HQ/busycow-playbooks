---
name: packaging-to-github
description: Use when publishing skills, agent setups, schemas, knowledge bases, or capability docs to the busycow-playbooks GitHub repo — so any agent or client can install them. Covers universalization (strip internal data), sanitization scan, repo structure, and push workflow.
triggers:
  - "push to github"
  - "publish playbook"
  - "package for clients"
  - "put on github"
  - "幫客戶裝"
  - "publish skill"
  - "push 上去"
  - "打包能力"
  - "分享設定"
version: "3.0"
author: BusyCow
---

# Packaging to GitHub

Use this when publishing **anything reusable** to `busycow-playbooks`:
- **Skills** — agent procedural memory (SKILL.md + scripts + references)
- **Setup** — Hermes profiles, SOUL.md, MEMORY.md, USER.md templates
- **Schema** — Lark Bitable table/field definitions (SCHEMA.md)
- **Knowledge** — GBrain pages, decision logs, architecture docs
- **Capability Docs** — agent capability documents (what an agent can do, how to trigger it)

## What busycow-playbooks Is

`busycow-playbooks` is the **universal agent team template** — a reusable blueprint for deploying the full DataXquad agent stack at any company.

It is built to be copied. When onboarding a new client or spinning up a new company instance, you pull from this repo and fill in the company-specific layer. The playbook itself stays generic — no internal data, no hardcoded IDs, no product names.

**What it contains:**
- Agent CAPABILITIES docs — what each agent owns, how to trigger it, authority grid, skills and crons
- `context/` — structural data (CRM schema) and contextual knowledge (ICP, pricing, brand) in universalized form
- `shared-skills/` — Hermes skills every agent profile needs
- `standards/` — the canonical CAPABILITIES doc format (S1–S5)
- `third-party-tools/` — setup guides for Lark, Google, Cloudflare, Tavily
- `SETUP.md` — Hermes + GBrain core config

**How agents contribute:**
Each agent (Leo, Maya, Vera, etc.) works with their own `agent-teams/<agent>/` directory. When they build or update a Capability, they push directly to this repo. The workflow is:
1. Agent writes/updates `agent-teams/<agent>/CAPABILITIES.md` locally
2. Agent pushes to `busycow-playbooks` via `git push`
3. Iris (or the agent) runs `gbrain sync --repo ~/busycow-playbooks` to update the knowledge layer

**What agents do NOT push here:**
- Company-specific data (client names, real pricing, internal decisions) → goes to `<company>-core`
- Live credentials or API keys → never committed anywhere
- Task state or session output → nowhere (ephemeral)

---

## Repos

| Repo | Visibility | Purpose |
|---|---|---|
| `DataXquad-HQ/busycow-playbooks` | **Private** | Universal agent team template — capabilities, skills, standards, context |
| `DataXquad-HQ/dataxquad-core` | **Private** | DataXquad company knowledge — strategy, OKRs, ICP, decisions, market intel |
| `DataXquad-HQ/aquaoptima-core` | **Private** | AquaOptima company knowledge — product, market, partners, strategy, team |

**Key architectural rule — two repo types, different purposes:**

| Type | Repo pattern | What goes here | What does NOT go here |
|---|---|---|---|
| Playbook | `busycow-playbooks` | Agent CAPABILITIES docs (universalized), setup SOPs, shared skills, schemas | Company-specific data |
| Company core | `<company>-core` | Company strategy, ICP, market intel, partner registry, team, decisions | Agent design docs |

**Agent CAPABILITIES docs belong in `busycow-playbooks/agent-teams/<agent>/CAPABILITIES.md`** — not in company-core repos. They describe what the agent system can do, independent of any specific company's data. Company-core `agents/` directories should only contain a `README.md` pointing to the playbooks repo.

**Why:** A capability doc describes the agent design (triggers, skills, authority grid). It applies to any company running the same agent team. A company-core repo holds the specific data that fills those agents' context (ICP, pricing, decisions). Mixing them means the design doc gets contaminated with company data and can't be reused.

**Local paths:**
```
~/busycow-playbooks      ← agent design + setup playbooks (private)
~/dataxquad-core         ← DataXquad company knowledge (private)
~/aquaoptima-core        ← AquaOptima company knowledge (private)
```

### busycow-playbooks agent-teams structure
```
busycow-playbooks/
└── agent-teams/
    ├── README.md              ← roster + delegation map
    ├── iris/CAPABILITIES.md   ← ✅ v1.0
    ├── maya/CAPABILITIES.md   ← ✅ v2.0
    ├── leo/CAPABILITIES.md    ← ✅ v2.0
    ├── vera/CAPABILITIES.md   ← ✅ v1.0 (Partner Success Agent)
    ├── quinn/                 ← pending
    ├── rex/                   ← pending
    └── steve/                 ← pending
```

### dataxquad-core structure
```
dataxquad-core/
├── agents/README.md     ← POINTER ONLY — links to busycow-playbooks/agent-teams/
├── strategy/            ← OKRs, positioning, pricing rationale
├── context/             ← ICP definitions, market maps, competitor intel
├── decisions/           ← YYYY-MM-DD-topic.md decision logs
└── structural/          ← schemas, field definitions, data models
```

**Doc routing rule:**
- `busycow-playbooks` = agent design (CAPABILITIES, SOUL, skills) — universalized, reusable
- `<company>-core` = company context (strategy, ICP, market, partners, decisions)
- Lark = task board, IM, external-facing docs, financial forecasts with formulas
- Google = spreadsheets with formulas, external client/investor docs

**How agents read GitHub content:**
Agents do NOT `git clone` repos directly. The correct read path is:
1. Repos are registered as GBrain sources and synced: `gbrain sync --repo ~/busycow-playbooks`
2. Agents query via `mcp_gbrain_query(query="...", source_id="busycow-playbooks")`
3. Or direct `cat ~/busycow-playbooks/agent-teams/maya/CAPABILITIES.md` via terminal

**GBrain sources registered:**
- `busycow-playbooks` → `~/busycow-playbooks` (federated: true)
- `dataxquad-core` → `~/dataxquad-core` (federated: true)
- `aquaoptima-core` → `~/aquaoptima-core` (federated: true)

**Update flow for CAPABILITIES docs:** Write/update in `~/busycow-playbooks/agent-teams/<agent>/CAPABILITIES.md` → `git push` → `gbrain sync --repo ~/busycow-playbooks`. Do NOT put capabilities docs in company-core repos.

## References

- `references/agent-architecture-principles.md` — design principles for SOUL.md, agent operating models, three-layer capability structure, hire framework, event-driven triggers. Load when designing new agent playbooks or SOUL.md templates.

---

## What Gets Packaged — Decision Table

| Artifact type | Source location | Target in repo | Notes |
|---|---|---|---|
| Env key management | `~/.hermes/shared.env` + `sync-shared-env.sh` | `core/hermes/` | shared.env.example + profile.env.example + script — NEVER commit real .env |
| Skill (SKILL.md + support files) | `~/.hermes/skills/<cat>/<skill>/` | `shared-skills/<skill>/` | Copy entire folder — SKILL.md + scripts/ + references/ + templates/. Skills are **folders**, not flat .md files |
| Agent-specific skill | `~/.hermes/profiles/<agent>/skills/<skill>/` | `agent-teams/<agent>/skills/<skill>/` | Same folder structure; universalize before pushing |
| SOUL.md | `~/.hermes/profiles/<name>/SOUL.md` | `agent-teams/<agent>/SOUL.md` | Strip product/client names |
| MEMORY.md / USER.md | `~/.hermes/profiles/<name>/memories/` | `core/templates/` | Strip all specifics |
| CRM / DB schema | From SCHEMA.md or structural definition | `structural-data/<system>/SCHEMA.md` | Replace all IDs with `{{PLACEHOLDERS}}`; keep field types + relation patterns |
| Background knowledge | Prose docs (company, products, glossary) | `contextual-knowledge/<topic>.md` | Strip entity-specific intel; keep structural patterns |
| GBrain knowledge page | GBrain `put_page` export | `contextual-knowledge/<slug>.md` | Strip entity-specific takes/timelines; keep design principles |
| Third-party tool install | Docker Compose + README | `third-party-tools/<tool>/` | README.md + SETUP.md + docker-compose.yml (template form) |
| Core tool doc | Per-tool README | `core/<tool>/README.md` | What it is, how it integrates, setup steps |
| Agent capability doc | Lark Doc or local file | `agent-teams/<agent>/CAPABILITY.md` | Strip internal IDs and product names |

## Making a Skill Shared Across All Agent Profiles

Skills in `_shared/` are symlinks to `/mnt/disks/data/hermes/skills/` (the default profile skills folder). One source, all profiles inherit automatically. Updates to the source propagate instantly — no re-copy needed.

```bash
SKILL_SRC="/mnt/disks/data/hermes/skills/<category>/<skill-name>"
for agent in leo maya quinn rex; do
  ln -sf "$SKILL_SRC" ~/.hermes/profiles/$agent/skills/_shared/<skill-name>
  echo "$agent: linked ✓"
done
```

Verify:
```bash
ls -la ~/.hermes/profiles/maya/skills/_shared/<skill-name>
# Should show: -> /mnt/disks/data/hermes/skills/<category>/<skill-name>
```

**Do NOT copy the skill folder into each profile** — symlinks only. Copies break the single-source-of-truth model and require manual sync on every update.

Skills in `_shared/` are loaded by every agent automatically on each session. Use symlinks — not copies — so updates to the source propagate instantly. Verify with `ls -la ~/.hermes/profiles/<agent>/skills/_shared/<category>/`.

---

## Architecture Principles

**Three core pillars** — every deployment runs all three:
1. **Lark / Feishu** — workspace comms, databases (Bitable), documents
2. **Google Workspace** — external email, calendar, Drive, Sheets, Docs
3. **GBrain** — long-term knowledge graph (entities, decisions, intel)

**Tasks are business-anchored.** No standalone Task Tracker. Tasks live inside the business Base (Sales & Ops, Finance, etc.) and link via DuplexLink to Opportunities/Partnerships/Initiatives. This is a hard architectural rule — do not propose a separate Task Tracker.

**GBrain setup includes system prompt refinement.** Installing GBrain also means appending routing rules to `~/.hermes/SOUL.md` so the agent auto-writes entities/decisions to GBrain without being asked.

## Knowledge Repos (company-specific, private)

For company-specific private knowledge bases (not the public playbooks repo), use this pattern:

**Repos created:**
- `DataXquad-HQ/dataxquad-core` — DataXquad company core (~/dataxquad-core)
- `DataXquad-HQ/aquaoptima-core` — AquaOptima company core (~/aquaoptima-core)

**Canonical directory structure:**
```
<company>-core/
├── README.md           ← repo purpose + reading order
├── product/            ← what the product is
├── market/             ← ICP, expansion roadmap
├── partners/           ← partner strategy + registry
├── strategy/           ← company strategy, OKRs, milestones
├── team/               ← roster, RACI, entity info
├── operations/         ← live sites, SLA, deployment
└── decisions/          ← key decisions (YYYY-MM-DD-topic.md)
```

Files are tagged `v0.1 — team to update` on first creation. Each file is standalone markdown — no cross-file dependencies. Agents read these directly via `cat` or via GBrain sync.

## Repo Structure (canonical as of 2026-06-12)
```
busycow-playbooks/
├── README.md
├── SETUP.md                    ← Hermes + GBrain core config
├── github-as-core-knowledge.md ← SOP for using GitHub as knowledge layer
├── context/                    ← structural + contextual data
│   ├── README.md
│   ├── structural/
│   │   └── crm.md              ← Twenty CRM objects + fields + relations
│   └── contextual/
│       ├── 01-company-overview.md
│       ├── 02-products.md
│       ├── 03-target-customer-icp.md
│       ├── 04-pricing.md
│       ├── 05-pic.md
│       └── 06-brand-messaging.md
├── third-party-tools/          ← platform tools (Lark, Google, Cloudflare, Tavily)
│   ├── cloudflare-tunnels/README.md
│   ├── google-workspace/README.md
│   ├── lark/README.md
│   └── tavily/README.md
├── shared-skills/              ← Hermes skills for ALL profiles (folder-per-skill)
│   ├── capturing-to-gbrain/
│   ├── creating-shared-skill/
│   ├── github-core-repos/
│   └── ...
├── standards/
│   └── agent-capability-doc-standard.md  ← canonical S1–S5 format for CAPABILITIES.md
└── agent-teams/                ← per-agent CAPABILITIES docs (universalized)
    ├── README.md               ← roster + delegation map
    └── <agent>/                ← iris / leo / maya / vera / quinn / rex / steve
        ├── CAPABILITIES.md
        └── context/            ← agent-specific structural + contextual files
```

**context/ folder rules:**
- `context/structural/` — data schemas the agent reads/writes (CRM objects, field definitions)
- `context/contextual/` — knowledge the agent uses to make judgments (ICP, pricing, principles, product context)
- Both are flat md files — no sub-subfolders
- No duplicate content between `context/structural/` and `context/contextual/` — each file has one home

---

## Workflow

### Step 1 — Identify what to package

Before starting, classify the artifact:

**Skill** → `skill_view(name)` to get full content + linked files  
**SOUL.md / MEMORY.md / USER.md** → `read_file(~/.hermes/profiles/<name>/SOUL.md)`  
**Schema** → query Lark Bitable field definitions via `lark-cli base` or read existing SCHEMA.md  
**GBrain knowledge page** → `mcp_gbrain_get_page(slug)` to export  
**Capability Doc** → `read_file` or fetch from Lark Doc  

### Step 2 — Universalize

Apply these rules in ORDER (broader patterns first):

| Remove | Replace with |
|--------|-------------|
| Hardcoded Lark base/table IDs (`tbl[A-Za-z0-9]{8,}`) | `{{TABLE_ID}}` |
| Hardcoded field IDs (`fld[A-Za-z0-9]{8,}`) | `{{FIELD_ID}}` |
| User open_ids (`ou_[a-f0-9]{20,}`) | `{{USER_OPEN_ID}}` |
| API tokens (Lark, Notion, Google) | `{{API_TOKEN}}` or remove |
| Google Doc / Drive IDs | `{{GOOGLE_DOC_TEMPLATE_ID}}` |
| IP addresses | `{{SERVER_IP}}` |
| Internal product names (AquaOptima, GeoKernel, TRACI) | `[Product]` or `[your product lines]` |
| Specific client/partner names (HKRFID, Onnet, etc.) | `[Client]` or `[Partner]` |
| Personal names / usernames (Hunter, hunter_lin) | `the owner` or omit |
| Personal file paths (/home/username, /mnt/disks/data/hermes) | `~/.hermes` |
| Internal business logic (billing entity rules, commission %) | Generic equivalent or remove |
| Company name DataXquad | BusyCow (as publisher) |
| Lark App ID (`cli_[a-z0-9]{16,}`) | `{{LARK_APP_ID}}` |
| Lark App Secret | `{{LARK_APP_SECRET}}` |

**Schema-specific rules:**
- Replace every table name that's product-specific → `[Module Name]` (keep structural names like Tasks, Activities, Contacts)
- Replace every option value that's business-specific (e.g. `AquaOptima`, `GeoKernel`) → `[Value]`
- Keep field types, link directions, and relationship patterns — these are the reusable structural knowledge

**GBrain knowledge page rules:**
- Remove entity-specific takes, timelines, and fact rows that reference internal clients/products
- Keep decision rationale, design principles, and architectural patterns — these generalize
- Remove `## Facts` fences entirely if they contain specifics
- Keep `## Summary`, `## Key Principles`, `## How It Works` sections

**Capability Doc rules:**
- Keep the capability name, trigger description, input/output format, and workflow steps
- Replace specific Lark base/table references with `{{PLACEHOLDER}}`
- Remove example data that references real clients or deals

**Key insight:** Apply broad patterns (`tbl[A-Za-z0-9]{8,}`, `ou_[a-f0-9]{20,}`) FIRST in code, BEFORE specific named replacements. Broad patterns catch everything; named patterns handle remaining context.

### Step 3 — Sanitization scan (mandatory before push)

```python
import re, os

CONFIDENTIAL_PATTERNS = [
    (r'tbl[A-Za-z0-9]{8,}', 'hardcoded table ID'),
    (r'fld[A-Za-z0-9]{8,}', 'hardcoded field ID'),
    (r'ou_[a-z0-9]{20,}', 'hardcoded open_id'),
    (r'oc_[a-z0-9]{20,}', 'hardcoded chat ID'),
    (r'ntn_[A-Za-z0-9]+', 'Notion token'),
    (r'LARK_APP_(?:ID|SECRET)=[^\n{]', 'Lark credential'),
    (r'MtvNbgCH[A-Za-z0-9]+', 'hardcoded Lark app token (Sales CRM)'),
    (r'SfMJbDOB[A-Za-z0-9]+', 'hardcoded Lark app token (Task Tracker)'),
    (r'529a4913-cc22-4e1b-b8ee-52a53c4c5d3c', 'Twenty API Key ID'),
    (r'a352ccf9-ed5f-40d3-910f-706074dc3877', 'Twenty Workspace ID'),
    (r'Iris@DataXquad2026', 'Twenty admin password'),
    (r'hunter\.lin@distify\.ai', 'internal admin email'),
    (r'100\.118\.240\.101', 'Tailscale IP (use {{SERVER_TAILSCALE_IP}})'),
    (r'\bDataXquad\b(?!-HQ)', 'company name DataXquad'),
    (r'\b(AquaOptima|GeoKernel|TRACI)\b', 'internal product name'),
    (r'/home/[a-z_]+', 'personal path'),
    (r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b', 'IP address'),
    (r'[a-z]+@dataxquad\.com', 'internal email'),
    (r'\bhunter_lin\b', 'personal username'),
    (r'\b(HKRFID|AICities|Onnet|HeadWorker|VoiceBot)\b', 'internal partner name'),
]

def scan(directory):
    issues = {}
    for root, _, files in os.walk(os.path.expanduser(directory)):
        if '.git' in root:
            continue
        for filename in files:
            if not filename.endswith('.md'):
                continue
            filepath = os.path.join(root, filename)
            with open(filepath) as f:
                content = f.read()
            file_issues = []
            for pattern, label in CONFIDENTIAL_PATTERNS:
                matches = re.findall(pattern, content)
                if matches:
                    file_issues.append(f"  ⚠️  {label}: {list(set(matches))[:3]}")
            if file_issues:
                issues[filepath] = file_issues
    return issues

issues = scan("~/busycow-playbooks")
if not issues:
    print("✅ All clean — safe to push.")
else:
    for f, problems in issues.items():
        print(f"📄 {f}")
        for p in problems:
            print(p)
```

**Known false positives to ignore:**
- `LARK_APP_SECRET=***` in README Step 0c — example text, not a real secret
- English words starting with `rec` — NOT Lark record IDs

**Do not push if scan returns any real issues. Fix first.**

### Step 4 — Write files to repo

Save each universalized file to the correct location under `~/busycow-playbooks/`.

**Skills with support files:** Each internal skill is a folder. Copy the whole directory:
```bash
cp -r ~/.hermes/skills/<cat>/<skill>/ ~/busycow-playbooks/shared-skills/<skill>/
```
Then universalize every file inside (SKILL.md + references/ + scripts/).

**Skills are NEVER flat .md files in the repo** — always folders, even if the only file is SKILL.md:
```
shared-skills/twenty-crm/SKILL.md       ✅
shared-skills/twenty-crm.md             ❌  (old flat format, do not use)
```

**GBrain knowledge pages:** Export via `mcp_gbrain_get_page`, universalize, write to `playbooks/<domain>/knowledge/<slug>.md`. Add a `## Install Note` section at the top explaining how to import: `gbrain import <file>` or manually via `put_page`.

**Capability Docs:** Write to `playbooks/agents/profiles/<agent>/CAPABILITY.md`. Format:
```markdown
# <Agent Name> — Capability Document

## What This Agent Does
<one-paragraph description of role and scope>

## Capabilities
| # | Capability | Trigger | Output |
|---|---|---|---|
| 1 | <Name> | <how to invoke> | <what it produces> |

## Input Requirements
<what data/context the agent needs to operate>

## Output Standards
<format and quality standards for agent output>
```

### Step 5 — Push
```bash
cd ~/busycow-playbooks
git add -A
git commit -m "feat/refactor: [description of what was added or changed]"
git push origin main
```

### Step 6 — Confirm client URLs
```
https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/[path]/SETUP.md
```
After push: `curl -s [raw URL] | head -5` to verify accessible.

---

## Installation Order (client-facing)

```
Phase 0: Human steps (manual)
  1. VM + Hermes install
  2. Lark org + Hermes App
  3. Lark Dev Console → enable permissions (messenger / base / docs / contact)
  4. hermes setup lark → get Lark user ID approved
  5. Tavily API key → hermes config set search.tavily_api_key YOUR_KEY
  6. (optional) hermes setup google-workspace

Phase 1: setup/SETUP.md      → SOUL.md + MEMORY.md + USER.md
Phase 2: core/SETUP.md       → GBrain + Lark MCP + Google Workspace + Hermes Registry
Phase 3: playbooks/X/SETUP.md → business Base schema + skills + chat groups + cron jobs
```

---

## SETUP.md Format (agent-executable)

Every SETUP.md must be written as **step-by-step agent instructions**, not human prose.

Required sections:
1. What this setup creates (bullet list)
2. Numbered steps with exact API calls or fetch URLs
3. Verify section (how to confirm it worked)
4. Next step (what to run next)

Must be **idempotent** — running twice should not corrupt the workspace.

---

## Multi-Agent Playbook Structure

When packaging a full multi-agent system, use the `playbooks/agents/` structure:

```
playbooks/agents/
├── README.md         ← agent roster table, design philosophy
├── SETUP.md          ← 10-step agent-executable install
├── SCHEMA.md         ← full structural data definition (all tables, fields, links)
├── knowledge/        ← universalized GBrain pages
│   └── agent-operating-model.md
└── profiles/
    ├── default/
    │   ├── SOUL.md
    │   └── CAPABILITY.md
    ├── leo/
    │   ├── SOUL.md
    │   └── CAPABILITY.md
    └── ...
```

**SOUL.md template rules:**
- Never include specific deal names, client names, or product names → `[Product]`, `[Client]`, `[Partner]`
- Never include Lark table IDs or App Tokens → `{{LARK_APP_TOKEN}}`, `{{TABLE_ID_TASKS}}`
- Always include: Role, How You Work, Authority & Boundaries, GBrain Access level, Tools You Rely On, Lark Bitable Access section

**SCHEMA.md structure:** 5 layers — Strategy (Goals → Initiatives), Execution (Tasks), CRM (Accounts, Contacts, Activities), Deals (Opportunities, Partnerships), Commercial Documents (Quotations & Invoices).

---

## Parallelising Large Batches

**Do NOT use `delegate_task` for bulk packaging** — subagents time out (~80s) on large reads.

**Use `execute_code` with a batch Python script instead:**
```python
skill_mapping = {
    'core/skills/hermes-agent.md': 'core/skills/hermes-agent.md',
    # ... all mappings
}
for dest_rel, src_rel in skill_mapping.items():
    content = read_skill(os.path.join(SKILLS_ROOT, src_rel))
    content = universalize(content)
    os.makedirs(os.path.dirname(dest_path), exist_ok=True)
    with open(dest_path, 'w') as f:
        f.write(content)
```

---

## Pitfalls

- **`gh` CLI is not available on this VM** — do not attempt `gh repo create`. Use the **Python script pattern** (more reliable than shell heredocs which break on JSON with special chars):
  ```python
  # /tmp/create_repo.py
  import urllib.request, json
  env = {}
  with open("/mnt/disks/data/hermes/.env") as f:
      for line in f:
          if "=" in line and not line.startswith("#"):
              k, v = line.split("=", 1)
              env[k.strip()] = v.strip()
  token = env.get("GITHUB_TOKEN", "")
  req = urllib.request.Request(
      "https://api.github.com/orgs/DataXquad-HQ/repos",
      data=json.dumps({"name": "repo-name", "private": True, "auto_init": True}).encode(),
      headers={"Authorization": f"Bearer {token}", "Accept": "application/vnd.github+json",
               "X-GitHub-Api-Version": "2022-11-28", "Content-Type": "application/json"},
      method="POST"
  )
  with urllib.request.urlopen(req) as resp:
      d = json.loads(resp.read())
      print("URL:", d.get("html_url")); print("private:", d.get("private"))
  ```
  Run: `python3 /tmp/create_repo.py`. Then: `git clone git@github.com:DataXquad-HQ/repo-name.git` (SSH, not HTTPS).
- **Making an existing repo private** — same Python pattern but PATCH to `/repos/<org>/<repo>` with `{"private": True}`. Shell heredoc approach fails with EOF/quote errors when the .env source is inline.
- **SSH shared across all agent profiles** — all profiles run under the same Linux user (`hunter_lin`), so `~/.ssh/config` and keys are automatically shared. No per-profile setup needed. Key in use: `~/.ssh/github_geokernel` pointing to `github.com`. Verify: `ssh -T git@github.com` → `Hi hunterlin1997!`. Only needs attention when adding a new VM or new Linux user.
- **HTTPS clone + SSH push → fatal: could not read Username** — `git clone https://github.com/...` sets the remote URL to HTTPS. Pushing requires credentials (GitHub removed password auth). Fix once with: `git remote set-url origin git@github.com:DataXquad-HQ/busycow-playbooks.git` — subsequent pushes use the SSH key already configured on the VM (`hunterlin1997` key). Verify SSH works first: `ssh -T git@github.com`.
- **Established knowledge repo structure** — `dataxquad-core` and `aquaoptima-core` use the same pattern: `README.md` + subdirs (`product/`, `market/`, `partners/`, `strategy/`, `team/`, `operations/`, `decisions/`). Each file is a standalone markdown knowledge doc tagged `v0.1 — team to update`. This is the canonical structure for any company-level knowledge repo.
- Run sanitization scan **every time**, even for small edits
- `{{PLACEHOLDER}}` format for all dynamic values — don't leave bare TODO comments
- SETUP.md must work as a **standalone agent instruction** — don't assume the agent has context from the README
- Raw GitHub URLs only for client install — `raw.githubusercontent.com`, not the HTML page
- Don't add Notion-based variants or other stacks — one stack only (Hermes + Lark + GBrain)
- After push, verify with: `curl -s [raw URL] | head -5`
- Sanitization scan: use `os.path.expanduser()` — bare `~` doesn't expand in Python's `os.walk`
- Internal server paths appear in two forms: `/home/the_owner/` AND `/mnt/disks/data/hermes/` — use greedy pattern: `r'/mnt/disks/data/hermes[^\s]*'` → `~/.hermes`
- Scan false positives: `LARK_APP_SECRET=***` in README is safe (example text)
- **SOUL.md templates must be universalized** — strip all specific deal/client/product names
- **GBrain pages:** only export structural patterns and decisions, never entity-specific timelines or fact rows
- **Capability Docs:** the trigger description is the most valuable part — make it concrete and specific
- **shared-skills/ files must be fully universalized** — all credentials/IDs use `{{PLACEHOLDER}}` format; the twenty-crm.md skill is the reference example
- **structural-data/ vs agent-teams/**: cross-agent schemas (CRM object definitions, DB schemas) go in `structural-data/`, NOT inside any agent's directory; agent-teams/ is for SOUL.md, capabilities, and agent-specific skills only
- **Remote URL**: repo is at `git@github.com:DataXquad-HQ/busycow-playbooks.git` — GitHub may still redirect from old `BusyCow-DataXquad` org; update remote after clone with `git remote set-url origin git@github.com:DataXquad-HQ/busycow-playbooks.git`
