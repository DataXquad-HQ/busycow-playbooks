# Agents Playbook — Setup

This file is written for agent execution. Read each step and execute it in order. Do not skip steps. The setup is idempotent — running it twice will not corrupt an existing installation.

## What This Creates

- Six Hermes profiles: `default` (Iris), `steve`, `leo`, `quinn`, `maya`, `rex`
- SOUL.md and MEMORY.md for each profile
- Shared skills symlinked into all profiles
- Role-specific skills per profile
- GBrain + Lark MCP configured in each profile's config.yaml
- Cron jobs: Iris daily dispatcher, nightly GBrain dream, daily task briefing, weekly audit
- Lark Bitable schema: Goals, Initiatives, Tasks, Accounts, Contacts, Activities, Opportunities, Partnerships, Quotations & Invoices
- Tavily configured as the web search backend

---

## Prerequisites

Before running this setup, the following must already be complete:

- Phase 0 (VM + Hermes installed) ✓
- Phase 1 (setup/SETUP.md) ✓ — SOUL.md, MEMORY.md, USER.md written
- Phase 2 (core/SETUP.md) ✓ — GBrain, Lark MCP, Google Workspace, core skills installed

Additionally, obtain before proceeding:
- Tavily API key (register at tavily.com)
- LLM API key (OpenAI preferred; Anthropic or Google accepted)

---

## Step 0: Configure Tavily

Tavily is the web search backend used by all agent profiles.

```bash
# Set Tavily API key in Hermes config
hermes config set search.tavily_api_key {{TAVILY_API_KEY}}

# Verify
hermes config get search.tavily_api_key
```

---

## Step 1: Create Agent Profiles

Create each profile. If a profile already exists, this is safe to skip.

```bash
hermes profile create steve
hermes profile create leo
hermes profile create quinn
hermes profile create maya
hermes profile create rex
# Note: 'default' profile already exists (Iris)
```

---

## Step 2: Configure LLM and Lark MCP for All Profiles

All profiles use the same LLM provider and connect to the same Lark MCP server.

```bash
for PROFILE in default steve leo quinn maya rex; do
  # Set LLM provider (OpenAI preferred; change to anthropic or google if needed)
  hermes --profile $PROFILE config set model.provider openai
  hermes --profile $PROFILE config set model.default gpt-4o

  # Connect Lark MCP
  hermes --profile $PROFILE config set mcp_servers.lark.command "$(which lark-mcp)"
  hermes --profile $PROFILE config set mcp_servers.lark.args '["mcp", "--lark-app-id", "{{LARK_APP_ID}}", "--lark-app-secret", "{{LARK_APP_SECRET}}"]'
done
```

---

## Step 3: Connect All Profiles to GBrain

All profiles must be able to query the GBrain knowledge graph.

```bash
GBRAIN_MCP_CMD="$(which gbrain) mcp serve"

for PROFILE in default steve leo quinn maya rex; do
  hermes --profile $PROFILE config set mcp_servers.gbrain.command "$GBRAIN_MCP_CMD"
done
```

Verify the connection from any profile:

```bash
hermes --profile default "Query GBrain: what pages exist? Use mcp_gbrain_get_stats."
```

---

## Step 4: Connect All Profiles to Google Workspace

All profiles share the same Google OAuth token. The credential file must already exist from Phase 2.

```bash
for PROFILE in default steve leo quinn maya rex; do
  # Install Google Workspace skill into each profile
  hermes --profile $PROFILE skill install google-workspace
done
```

The skill reads the token from `~/.hermes/google_token.json` automatically. No per-profile re-authentication is needed.

---

## Step 5: Install Shared Skills

These skills are used by all agents. Install them once into the default profile and symlink into all others.

```bash
SHARED_SKILLS=(
  "lark-base"
  "lark-im"
  "lark-task"
  "lark-doc"
  "lark-drive"
  "lark-calendar"
  "lark-contact"
  "lark-shared"
  "lark-workflow-standup-report"
  "reading-lark-files"
  "managing-tasks"
  "reviewing-tasks"
  "planning-next-actions"
  "capturing-to-gbrain"
  "hermes-agent"
  "managing-cron-jobs"
)

for SKILL in "${SHARED_SKILLS[@]}"; do
  hermes skill install $SKILL
  for PROFILE in steve leo quinn maya rex; do
    hermes --profile $PROFILE skill link $SKILL
  done
done
```

---

## Step 6: Write SOUL.md for Each Profile

Fetch the SOUL.md template for each profile and install it.

```bash
BASE_URL="https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/agents/profiles"

for PROFILE in default steve leo quinn maya rex; do
  curl -fsSL "${BASE_URL}/${PROFILE}/SOUL.md" > ~/.hermes/profiles/${PROFILE}/SOUL.md
  echo "SOUL.md installed for ${PROFILE}"
done
```

After installing, open each SOUL.md and fill in any `{{PLACEHOLDER}}` values specific to your deployment (company name, product lines, key contacts).

---

## Step 7: Install Role-Specific Skills

### Iris (default) — Chief of Staff
```bash
hermes skill install maintaining-gbrain
hermes skill install extracting-lark-to-gbrain
hermes skill install auditing-tasks
hermes skill install generating-task-briefing
hermes skill install capturing-sales-intel
hermes skill install reviewing-sales-pipeline
```

### Steve — Development Lead
```bash
hermes --profile steve skill install graphify
hermes --profile steve skill install kanban-orchestrator
hermes --profile steve skill install kanban-worker
hermes --profile steve skill install designing-multi-agent-systems
hermes --profile steve skill install packaging-to-github
hermes --profile steve skill install diagnosing-github-ci-failures
```

### Leo — Chief Revenue Officer
```bash
hermes --profile leo skill install capturing-sales-intel
hermes --profile leo skill install enriching-leads
hermes --profile leo skill install managing-sales-pipeline
hermes --profile leo skill install managing-partnership-pipeline
hermes --profile leo skill install reviewing-sales-pipeline
hermes --profile leo skill install generating-quotations
hermes --profile leo skill install generating-invoices
hermes --profile leo skill install tracking-financials
hermes --profile leo skill install maps
hermes --profile leo skill install xurl
```

### Quinn — Researcher
```bash
hermes --profile quinn skill install running-strategic-council
hermes --profile quinn skill install graphify
hermes --profile quinn skill install building-gbrain-knowledge-graph
hermes --profile quinn skill install analysing-erp-requirements
hermes --profile quinn skill install building-investor-financial-model
hermes --profile quinn skill install numpy-financial
hermes --profile quinn skill install extracting-lark-to-gbrain
hermes --profile quinn skill install maintaining-gbrain
hermes --profile quinn skill install managing-team-knowledge
hermes --profile quinn skill install researching-utility-energy-costs
```

### Maya — Content & Growth
```bash
hermes --profile maya skill install writing-blog-post
hermes --profile maya skill install marp-pitch-deck
hermes --profile maya skill install humanizer
hermes --profile maya skill install baoyu-infographic
hermes --profile maya skill install claude-design
hermes --profile maya skill install sketch
hermes --profile maya skill install generating-decks-with-open-design
hermes --profile maya skill install lark-docx-writer
hermes --profile maya skill install youtube-content
hermes --profile maya skill install xurl
```

### Rex — Support Engineer
```bash
hermes --profile rex skill install support/remote-debug
hermes --profile rex skill install building-gbrain-knowledge-graph
hermes --profile rex skill install extracting-lark-to-gbrain
hermes --profile rex skill install managing-team-knowledge
hermes --profile rex skill install graphify
hermes --profile rex skill install numpy-financial
```

---

## Step 8: Create Lark Bitable Schema

This step creates the full Structural Data schema in Lark. Run the schema setup skill:

```bash
hermes "Read the schema at https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/playbooks/agents/SCHEMA.md and create all tables in a new Lark Bitable Base called '{{COMPANY_NAME}} — Ops'. Use the lark-bitable-schema-setup skill. After creating all tables, record the App Token and all table IDs."
```

After this completes, add the App Token and table IDs to each profile's MEMORY.md:

```bash
# Edit each profile's MEMORY.md to add:
# Lark Bitable App Token: {{APP_TOKEN}}
# Goals table: {{TABLE_ID_GOALS}}
# Initiatives table: {{TABLE_ID_INITIATIVES}}
# Tasks table: {{TABLE_ID_TASKS}}
# (and so on for each table)
```

---

## Step 9: Create Cron Jobs

```bash
# Iris Daily Dispatcher — 09:00 local time every day
hermes cron create \
  --name "Iris Daily Dispatcher" \
  --schedule "0 1 * * *" \
  --prompt "You are Iris, Chief of Staff. Scan the Task Board in Lark Bitable. For each pending task: (1) check if Depends On tasks are done, (2) inject relevant GBrain and Lark context as Handoff Context, (3) assign to the appropriate agent profile. Use managing-tasks and reviewing-tasks skills." \
  --skills "managing-tasks,reviewing-tasks,lark-base"

# GBrain Nightly Dream — 20:00 UTC every day
hermes cron create \
  --name "GBrain Nightly Dream" \
  --schedule "0 20 * * *" \
  --prompt "Run the GBrain nightly maintenance cycle: embed new pages, consolidate facts, auto-link via wikilinks, update brain score. Then sync memory files to GitHub." \
  --skills "maintaining-gbrain,syncing-brain-memory"

# Nightly Lark → GBrain Extract — 19:00 UTC every day
hermes cron create \
  --name "Nightly Lark to GBrain Extract" \
  --schedule "0 19 * * *" \
  --prompt "Extract yesterday's Lark group chat messages and distil key knowledge into GBrain." \
  --skills "extracting-lark-to-gbrain"

# Daily Task Briefing — weekdays 09:00 local time
hermes cron create \
  --name "Daily Task Briefing" \
  --schedule "0 1 * * 1-5" \
  --prompt "Generate today's task briefing and send it to the main Feishu channel." \
  --skills "generating-task-briefing"

# Weekly Task Audit — Sunday 09:00 local time
hermes cron create \
  --name "Weekly Task Audit" \
  --schedule "0 1 * * 0" \
  --prompt "Run the weekly task audit: check for orphan tasks, stale initiatives, missing owners, and grouping issues. Report findings." \
  --skills "auditing-tasks"
```

---

## Step 10: Verify

Run these checks to confirm everything is working.

```bash
# Check all profiles exist
hermes profile list

# Check GBrain is accessible from all profiles
for PROFILE in default steve leo quinn maya rex; do
  hermes --profile $PROFILE "Query GBrain stats. Use mcp_gbrain_get_stats."
done

# Check Lark MCP is working
hermes "Send a test message to me on Feishu saying 'Agents playbook setup complete.'"

# Check cron jobs
hermes cron list
```

---

## Next Steps

- Fill in all `{{PLACEHOLDER}}` values in each profile's SOUL.md and MEMORY.md
- Add your company's key contacts and accounts to GBrain: `people/` and `companies/` pages
- Run the Sales playbook if needed: `playbooks/sales/SETUP.md`
- For software development capability, run the optional dev stack setup: `playbooks/software-dev/SETUP.md`
