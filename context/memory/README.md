# Memory Layer

This folder documents the team's two-layer memory architecture: **Hindsight** (working memory) and **GBrain** (knowledge wiki).

| Document | What |
|---|---|
| [`hindsight.md`](./hindsight.md) | Deployment, bank structure, API operations, agent protocol, pitfalls |
| [`gbrain.md`](./gbrain.md) | GBrain structure, read/write patterns, sync rules, auto-write triggers |

## One-Line Rule

> **Hindsight first** for live context. **GBrain** for complete documents. Iris syncs important GBrain facts into Hindsight after every significant update.
