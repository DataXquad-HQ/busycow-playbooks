# GitHub as Core Knowledge — Architecture & Setup

**Last Updated:** 2026-06-12

---

## The Problem with Lark/Notion for Core Docs

Most teams default to putting core knowledge in their messaging platform — Lark docs, Notion pages, Confluence. This works for humans, but breaks for AI agents:

- Agents need auth flows and scope approvals to read a Lark doc
- Lark docs are not natively version-controlled
- Reading a Lark doc is 3–5 API calls; reading a markdown file is 0
- Agents can't grep, diff, or batch-read Lark docs without custom tooling
- When you change a doc, agents don't know it changed

**GitHub solves all of this.** A private repo with markdown files is the most agent-readable, version-controlled, auditable knowledge store available.

---

## The Architecture

```
GitHub (private repo)
    └── company-core/           ← markdown files, structured by domain
            ├── product/
            ├── market/
            ├── partners/
            ├── strategy/
            ├── team/
            └── decisions/
                    ↓
              GBrain sync
                    ↓
         All agents query via GBrain
         or direct `cat` via terminal
```

**Two access paths for agents:**

1. **GBrain** — repo is synced as a GBrain source. Agents query with `mcp_gbrain_query` and get semantically-matched chunks. Best for "what do I need to know about X" questions.
2. **Direct `cat`** — agent reads the file directly via terminal. Best for "give me the full strategy doc" or "show me the exact pricing table".

---

## Why This Beats Lark for Core Knowledge

| Criteria | Lark Doc | GitHub Markdown |
|---|---|---|
| Agent reads without auth | ❌ Requires API scope | ✅ `cat file.md` |
| Version history | ✅ (revision_id) | ✅ `git log` + full diff |
| Diffable changes | ❌ | ✅ `git diff` |
| GBrain sync | ❌ Manual | ✅ Auto on push |
| Human readable | ✅ | ✅ (rendered in GitHub) |
| Batch readable | ❌ | ✅ `find . -name "*.md"` |
| Offline / portable | ❌ | ✅ `git clone` |
| PR review for changes | ❌ | ✅ |

**Lark stays useful for:** task boards (Bitable), IM / messaging, meeting notes, external-facing docs with complex formatting.

---

## What Goes in the GitHub Core Repo

| Goes in GitHub | Stays in Lark |
|---|---|
| Agent Capabilities docs | Task Tracker (Bitable) |
| Company Strategy / OKRs | Daily chat / IM |
| ICP definitions | Meeting notes (auto-generated) |
| Market maps and competitor intel | Reports with complex layout |
| Partner strategy and registry | External-facing decks |
| Pricing rationale | Financial forecast (formula-heavy) |
| Key decisions (log) | Contracts / legal docs |
| Agent SOUL.md | Quotations / invoices |
| All reference docs agents read | |

**Rule of thumb:** If an agent needs to read it to do its job → GitHub. If a human needs to collaborate on it in real time → Lark.

---

## Repo Naming Convention

Each company or product gets its own private repo under the `DataXquad-HQ` org:

```
DataXquad-HQ/dataxquad-core       ← DataXquad parent company
DataXquad-HQ/aquaoptima-core      ← AquaOptima portfolio company
DataXquad-HQ/geokernel-core       ← GeoKernel portfolio company (when ready)
DataXquad-HQ/busycow-playbooks    ← BusyCow playbook (this repo — setup SOPs)
```

**Naming pattern:** `<company>-core` for company knowledge, `<product>-playbooks` for setup SOPs.

---

## Standard Repo Structure

Every `-core` repo follows this structure:

```
<company>-core/
├── README.md                  ← what's here + reading order for agents
├── product/
│   └── overview.md            ← what the product is, architecture, benchmarks
├── market/
│   ├── icp.md                 ← who the customer is, buying process
│   └── expansion-roadmap.md   ← target markets and timeline
├── partners/
│   └── partner-strategy.md    ← partner types, active registry, GTM motion
├── strategy/
│   └── company-strategy.md    ← milestones, revenue model, strategic bets
├── team/
│   └── team.md                ← roster, RACI, company entity
├── operations/
│   └── live-sites.md          ← deployments, SLA standards
└── decisions/
    └── YYYY-MM-DD-<topic>.md  ← one file per key decision
```

---

## Setting Up a New Core Repo

### Step 1 — Create the repo

```bash
# Via GitHub API (replace <name> and <description>)
curl -s -X POST \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/orgs/DataXquad-HQ/repos \
  -d '{"name":"<company>-core","description":"<description>","private":true,"auto_init":true}'
```

### Step 2 — Clone locally

```bash
git clone git@github.com:DataXquad-HQ/<company>-core.git ~/<company>-core
```

### Step 3 — Create standard structure

```bash
mkdir -p ~/<company>-core/{product,market,partners,strategy,team,operations,decisions}
```

### Step 4 — Write the knowledge

Start with these files in order:
1. `README.md` — what's here, reading order
2. `product/overview.md` — what the product is
3. `market/icp.md` — who the customer is
4. `strategy/company-strategy.md` — where it's going

### Step 5 — Connect to GBrain

```bash
# Register the repo as a GBrain source
gbrain sources add <company>-core --path ~/<company>-core --federated true

# Initial sync
gbrain sync --repo ~/<company>-core
```

Once synced, all agents that query GBrain will automatically surface content from this repo.

### Step 6 — Keep it updated

**Who updates it:** The team responsible for each company (e.g. AquaOptima team updates `aquaoptima-core`).

**Update discipline:**
- Any strategic decision → add a file in `decisions/YYYY-MM-DD-<topic>.md`
- Any market or product change → update the relevant file directly
- Agent outputs that contain new intel → Iris writes it to GBrain first, then a human promotes to the repo if it's durable enough to be core knowledge

**Commit convention:**
```
feat: add partner strategy for Indonesia
update: revise ICP buying process based on TWC engagement
decision: 2026-06-12 pricing model locked at $X/station/month
```

---

## Maintenance Discipline

**Core principle:** The repo is always truthful. If something in the repo is no longer accurate, update it immediately — don't let it drift.

**Who is responsible:**
- Iris (Chief of Staff agent) — flags when a core doc hasn't been updated in 14+ days after a strategic change
- Human (Hunter / Kevin) — confirms and merges updates
- Portfolio company teams — own their own `-core` repos

**Signal that a doc needs updating:**
- A decision was made in a conversation that contradicts the current doc
- A partner status changed
- A market target was revised
- A product capability was added or removed

---

## Pitfalls

- **Don't put credentials or tokens in core repos** — even private repos. Use `.env` files or secret managers for anything sensitive.
- **Don't make core repos public** — they contain ICP details, partner names, pricing rationale, and competitive intel.
- **Don't duplicate into Lark** — if it's in GitHub, it does not also need a Lark doc. Two sources of truth means neither is trusted.
- **Don't use branches for knowledge** — keep everything on `main`. Branches are for drafts; once merged, it's truth.
- **GBrain sync lag** — after a push, GBrain sync runs on schedule. For urgent context, agents can `cat` the file directly while waiting for sync.
