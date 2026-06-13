# Hindsight — Agent Memory Server

Hindsight is the team's shared working memory layer. Agents call it directly via REST API to store and retrieve facts, decisions, and context across sessions.

- **Benchmark**: 94.6% LongMemEval (vs Mem0 49%, Graphiti+FalkorDB 71.2%, Letta 83%)
- **License**: MIT
- **Deployment**: 1 Docker container + 1 Postgres container
- **Retrieval**: 4 parallel strategies — semantic, BM25 keyword, graph, temporal — fused via RRF

---

## Why Not a Hermes Memory Provider

Hindsight is **not** connected via `hermes memory setup`. Hermes memory providers are per-agent and session-bound. Hindsight is a shared team server that all agent profiles call independently.

Hermes config stays:
```yaml
memory:
  provider: ''   # empty = built-in flat file (MEMORY.md) only
```

Agents call Hindsight directly from SOUL.md–driven logic via `curl` / REST.

---

## Architecture

```
hindsight           (ghcr.io/vectorize-io/hindsight:latest-slim)
├── REST API  →  :8888  (internal only — do NOT expose via reverse proxy)
└── Web UI    →  :9999  (internal only)

hindsight-db        (pgvector/pgvector:pg16)
└── PostgreSQL + pgvector  (standalone — do NOT share with other app DBs)
```

**Why `latest-slim`?** 500MB vs 9GB for the full image. Slim offloads embeddings to OpenAI (already paid for) instead of bundling local models.

**Why standalone DB?** Other Postgres containers (Twenty, GBrain, Postiz) do not have `pgvector`. Migrations fail silently if run against a plain `postgres:16` image.

---

## Deployment

### Directory Layout

```
/opt/hindsight/          # or ~/hindsight/ — wherever your stack lives
├── docker-compose.yml
└── .env                 # secrets — never commit this file
```

### `.env`

```env
HINDSIGHT_API_LLM_PROVIDER=anthropic
HINDSIGHT_API_LLM_API_KEY=sk-ant-...
HINDSIGHT_API_LLM_MODEL=claude-haiku-4-5

HINDSIGHT_API_EMBEDDINGS_PROVIDER=openai
HINDSIGHT_API_EMBEDDINGS_OPENAI_API_KEY=sk-proj-...
HINDSIGHT_API_EMBEDDINGS_OPENAI_MODEL=text-embedding-3-small

HINDSIGHT_API_RERANKER_PROVIDER=rrf

HINDSIGHT_API_DATABASE_URL=postgresql://hindsight:hindsight_pass@hindsight-db:5432/hindsight
HINDSIGHT_API_HOST=0.0.0.0
HINDSIGHT_API_PORT=8888
```

> **Critical**: Use `env_file:` in docker-compose, never inline `environment:`.
> Anthropic keys contain `$` characters which Docker interpolates and corrupts when inlined.

### `docker-compose.yml`

```yaml
services:
  hindsight-db:
    image: pgvector/pgvector:pg16
    container_name: hindsight-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: hindsight
      POSTGRES_PASSWORD: hindsight_pass
      POSTGRES_DB: hindsight
    volumes:
      - hindsight_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hindsight"]
      interval: 5s
      timeout: 5s
      retries: 10

  hindsight:
    image: ghcr.io/vectorize-io/hindsight:latest-slim
    container_name: hindsight
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "8888:8888"
      - "9999:9999"
    volumes:
      - hindsight_data:/home/hindsight/.pg0
    depends_on:
      hindsight-db:
        condition: service_healthy

volumes:
  hindsight_db_data:
  hindsight_data:
```

### Start

```bash
docker compose up -d
# Verify
curl http://localhost:8888/health
# Expected: {"status":"healthy","database":"connected"}
```

---

## Bank Structure

A **bank** is Hindsight's isolation unit — a namespace with its own access boundary.
Banks are created once via `PUT` (idempotent).

### Standard Bank Layout (adapt names per workspace)

| Bank ID | Who Writes | Who Reads | Contains |
|---|---|---|---|
| `{prefix}-global` | Chief of Staff only | All internal agents | Company facts, product knowledge, approved decisions |
| `{prefix}-internal` | All internal agents | All internal agents | Active tasks, handoffs, team decisions, pipeline context |
| `{prefix}-agent-{name}` | That agent only | That agent only | Private working memory, session context, observations |
| `{prefix}-human-{name}` | Chief of Staff | Chief of Staff | Person persona, communication style, workload signals |

### Create a Bank

```bash
curl -X PUT http://localhost:8888/v1/default/banks/{bank_id} \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Human-readable name",
    "mission": "What this bank contains and who uses it."
  }'
```

### List Banks

```bash
curl http://localhost:8888/v1/default/banks
```

---

## API Operations

Base URL: `http://localhost:8888/v1/default/banks`

### Retain (store a memory)

```bash
POST /v1/default/banks/{bank_id}/memories
{
  "items": [
    {
      "content": "The fact, decision, or context to store.",
      "tags": ["tag1", "tag2"]
    }
  ]
}
```

### Recall (retrieve memories)

```bash
POST /v1/default/banks/{bank_id}/memories/recall
{
  "query": "What is the current status of X?",
  "top_k": 5
}
```

### Reflect (deep reasoning over accumulated memory)

```bash
POST /v1/default/banks/{bank_id}/memories/reflect
{
  "question": "What should the team focus on this week given current context?"
}
```

---

## Agent Memory Protocol

### When to call `retain`
- After every task completion → store outcome and key facts in `{prefix}-internal`
- After a key decision → store in `{prefix}-global` (if company-wide) or `{prefix}-internal`
- After interacting with a human → update their `{prefix}-human-{name}` bank
- After discovering new market/competitor intel → `{prefix}-global`

### When to call `recall`
- Start of every task → recall `{prefix}-internal` for current context
- Before assigning tasks → recall team workload and routing rules
- Before communicating with a human → recall their persona bank
- Any time context from a previous session is needed

### In SOUL.md

Add this block to each internal agent's SOUL.md:

```markdown
## Memory Protocol

**Before starting any task:**
recall(bank="{prefix}-internal", query="context for [task topic]")

**After completing any significant task:**
retain(bank="{prefix}-internal", content="[agent name] completed [task]. Result: [outcome]. Key facts: [anything future agents need to know]. Date: [today]")

**When discovering a company-wide fact:**
retain(bank="{prefix}-global", content="[fact]", tags=["[domain]"])

**Before responding to a human:**
recall(bank="{prefix}-human-{name}", query="[person's] current priorities and communication style")
```

---

## Pitfalls

| Problem | Cause | Fix |
|---|---|---|
| `ImportError: sentence-transformers` | `latest-slim` has no local models | Use `EMBEDDINGS_PROVIDER=openai` and `RERANKER_PROVIDER=rrf` |
| `Unknown reranker provider: none` | `none` is not a valid value | Use `rrf` (pure algorithm, no API key needed) |
| DB migration fails | Shared Postgres has no pgvector | Use standalone `pgvector/pgvector:pg16` container |
| API key corrupted at startup | Anthropic key contains `$` | Use `env_file:` — never inline `environment:` in compose |
| `curl` returns wrong data | API base is `/v1/default/` not `/api/v1/` | Correct base: `http://localhost:8888/v1/default/banks/` |
| Bank creation fails with 404/405 | Using `POST` to create banks | Banks are created with `PUT`, not `POST` |
| Hindsight starts before DB ready | Race condition | Use `depends_on: condition: service_healthy` |

---

## Resource Requirements

| Component | RAM | Storage | Cost |
|---|---|---|---|
| `hindsight` (slim) | ~1–1.5 GB | minimal | $0 |
| `hindsight-db` | ~500 MB | ~5 GB per 500K facts | $0 |
| OpenAI embeddings | — | — | ~$0.02 per 1M tokens |
| Anthropic LLM (retain/reflect) | — | — | ~$0.15 per 1M tokens (Haiku) |

At moderate agent activity (5–10 agents, daily use): estimated **under $10/month** total.

---

## Useful Endpoints

```bash
# Health check
GET  /health

# List all banks
GET  /v1/default/banks

# Bank detail
GET  /v1/default/banks/{bank_id}

# Metrics (Prometheus-compatible)
GET  /metrics

# Web UI
http://localhost:9999
```
