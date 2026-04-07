# Memory (Mirror)

Long-term semantic memory. Agents remember and recall by meaning, not keywords.

## Stack

- **API**: FastAPI on `:8844` (systemd: `mirror.service`)
- **Database**: Supabase PostgreSQL
- **Vectors**: pgvector extension, 1536 dimensions
- **Embeddings**: Gemini Embedding API (`gemini-embedding-001`, free)
- **Polisher**: Gemma 4 31B distills raw engrams into compressed facts (systemd timer)

## How It Works

```
Agent calls remember("Customer prefers morning meetings")
  → Mirror API receives text
  → Gemini embeds it → 1536-dim vector
  → Stored in Supabase with metadata (agent, project, timestamp)

Agent calls recall("customer meeting preferences")
  → Mirror API embeds the query
  → pgvector cosine similarity search
  → Returns top matches ranked by relevance
```

## Tenant Isolation

Each customer's memories are scoped by project:
- Context IDs prefixed: `{project}:context-id`
- Cross-tenant recall blocked at MCP layer (403)
- System tokens can read all memories

## Budget Tracking

`mirror/budget.py` — Supabase-backed cost tracking per agent:
- `record_cost(agent, provider, model, tokens, cost)`
- `check_budget(agent, cost) → {allowed, warning, remaining}`
- `get_usage_summary(agent) → monthly breakdown`

## MCP Tools

- `remember(text)` — store a memory
- `recall(query, limit)` — semantic search
- `memories(limit)` — list recent memories
