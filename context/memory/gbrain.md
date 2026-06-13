# GBrain — Knowledge Wiki

GBrain is the team's structured knowledge wiki. It stores long-form documents — runbooks, decisions, company strategy, market intel — in versioned Markdown files, indexed for semantic search.

---

## What GBrain Is

- A git-backed Markdown wiki with vector embeddings + keyword search
- Source of truth for **complete, long-form documents** (not live working memory)
- Queryable by all agents via the `mcp_gbrain_*` tools in Hermes
- Synced from GitHub repos (pull → index → searchable)

---

## GBrain vs Hindsight

These two systems are complementary, not competing.

```
GBrain                              Hindsight
────────────────────────────────    ──────────────────────────────────
Format:  Markdown documents         Format:  Structured facts
Query:   slug / keyword / vector    Query:   Semantic + graph + temporal
Written: humans or agents           Written: agents after every interaction
Lifespan: versioned (long-term)     Lifespan: living memory (updated over time)
Good for: complete runbooks,        Good for: "what just happened",
          decision full-text,                 "who owns X",
          technical docs,                     task handoffs,
          market strategy                     person personas
```

**Rule:**
- **Hindsight first** — for all agent daily needs (task context, person persona, latest decision summary)
- **GBrain if needed** — for complete runbooks, detailed technical docs, full decision history
- **GBrain → Hindsight sync** — important long-form docs get a distilled summary retained into Hindsight so semantic search can find them

---

## Namespace Structure

GBrain organises pages by slug prefix. Standard namespaces:

| Prefix | Content |
|---|---|
| `people/` | Person profiles (internal team, external contacts) |
| `companies/` | Company profiles (clients, partners, competitors) |
| `decisions/` | Key decisions (format: `decisions/YYYY-MM-DD-topic`) |
| `projects/` | Active project overviews |
| `market/` | Market intel, ICP, expansion maps |
| `concepts/` | Durable domain knowledge |
| `agents/` | Agent identity, capabilities, routing rules |
| `analysis/` | Research outputs, synthesis docs |

---

## Writing to GBrain

### From Hermes (agent)

```python
# Create or update a page
mcp_gbrain_put_page(
    slug="decisions/2026-06-14-hindsight-deployment",
    content="""---
title: Hindsight Deployment Decision
date: 2026-06-14
type: decision
---

# Decision: Deploy Hindsight as Team Memory

**Decision**: Use Hindsight (self-hosted) as shared working memory for all agents.
**Rationale**: 94.6% LongMemEval, MIT license, 1 container, 4-strategy retrieval.
**Alternatives rejected**: Mem0 (49%), Graphiti+FalkorDB (SSPL concern), Letta (per-agent default).
"""
)

# Add a timeline entry to an existing page
mcp_gbrain_add_timeline_entry(
    slug="companies/aquaoptima",
    date="2026-06-14",
    summary="Hindsight memory server deployed for AquaOptima agent team"
)

# Extract facts from a conversation turn
mcp_gbrain_extract_facts(
    turn_text="Kevin confirmed the seed round target is USD1M, closing Sep 2026.",
    entity_hints=["companies/aquaoptima"]
)
```

### From CLI (human or cron)

```bash
# Sync a GitHub repo into GBrain
gbrain sync --repo ~/busycow-playbooks

# Query GBrain
gbrain query "what do we know about TWC partnership?"

# Get a specific page
gbrain get decisions/2026-06-14-hindsight-deployment
```

---

## Reading from GBrain

### From Hermes (agent)

```python
# Semantic search
mcp_gbrain_query(query="AquaOptima Malaysia expansion strategy")

# Get specific page
mcp_gbrain_get_page(slug="projects/aquaoptima-seed-round")

# Get backlinks (what links to this page)
mcp_gbrain_get_backlinks(slug="companies/aquaoptima")

# Recent salience (what's been active lately)
mcp_gbrain_get_recent_salience(days=7)
```

### Routing rule for agents

```
Need full text of a decision?     → mcp_gbrain_get_page(slug="decisions/...")
Need a runbook / technical doc?   → mcp_gbrain_query(query="...")
Need live working context?        → Hindsight recall (faster, relational)
Need a person's current status?   → Hindsight recall dx-human-{name}
Need a historical decision?       → GBrain decisions/ namespace
```

---

## GBrain ↔ Hindsight Sync

GBrain is **not** auto-synced to Hindsight. The flow is manual and intentional:

```
GBrain (full doc)  →  agent reads  →  distil 3–5 key facts  →  Hindsight retain
```

**What gets synced:**
- New company profiles → retain key facts to `{prefix}-global`
- New decisions → retain decision summary to `{prefix}-global`
- Updated person profiles → retain to `{prefix}-human-{name}`
- Market intel → retain key signals to `{prefix}-global`

**What does NOT get synced:**
- Raw Lark chat extracts (noise)
- Archive / historical docs
- Full Markdown text (too verbose — distil first)

**Sync trigger:**
- When Iris adds an important GBrain page, she immediately retains the summary to Hindsight
- Weekly Sunday audit: scan GBrain for pages without a corresponding Hindsight fact

---

## GitHub Repos as GBrain Sources

GBrain can ingest content from GitHub repos. All core repos are registered as federated sources.

| Source ID | Repo | Content |
|---|---|---|
| `busycow-playbooks` | `DataXquad-HQ/busycow-agent-team` | Agent capabilities, SOPs, context docs |
| `dataxquad-core` | `DataXquad-HQ/dataxquad-core` | DataXquad company knowledge |
| `aquaoptima-core` | `DataXquad-HQ/aquaoptima-core` | AquaOptima company knowledge |

```bash
# After pushing changes to a repo, sync to GBrain immediately
gbrain sync --repo ~/busycow-agent-team
gbrain sync --repo ~/dataxquad-core
```

Nightly cron also runs `gbrain sync` for all registered sources.

---

## Auto-Write Rules for Agents

Iris (and all agents) should write to GBrain proactively — without being asked:

| Trigger | Action |
|---|---|
| New person or company mentioned | `put_page people/` or `companies/` |
| Partnership or deal stage changes | `add_timeline_entry` on company page |
| Key decision reached | `put_page decisions/YYYY-MM-DD-topic` |
| New market or competitor intel | `put_page market/` + `extract_facts` |
| Important conversation signal | `extract_facts` |

---

## Pitfalls

- **Never write raw session transcripts to GBrain** — that's what `sessions/` DB is for
- **Lark extracts are noise** — don't bulk-import them; only distil key facts
- **Edit files in GitHub, not directly in GBrain** for content that lives in a repo — edit → push → `gbrain sync`
- **GBrain is the read layer** — GitHub is the write source for repo-backed content
- **Always `git pull` before editing** a repo-backed file
- **`gbrain sync` after every push** if agents need the content immediately (nightly cron is otherwise the fallback)
