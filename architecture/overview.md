# Architecture Overview

## Three Data Layers

```
┌──────────────────────────────────────────────────────────┐
│                  CLOUDFLARE (Edge)                        │
│  KV ─── Token registry for remote auth                   │
│  Pages ─ Static sites (DNU, GAF, TROP, docs)             │
│  Workers ─ Bus bridge proxy, update server                │
│                                                          │
│  Public endpoints:                                       │
│    mcp.mumega.com → :6070 (MCP SSE/HTTP)                 │
│    api.mumega.com → :8060 (Squad Service)                │
│    bus.mumega.com → :6380 (Bus Bridge)                   │
└─────────────────────┬────────────────────────────────────┘
                      │ nginx reverse proxy
┌─────────────────────▼────────────────────────────────────┐
│                  HETZNER VPS                              │
│                                                          │
│  ┌──────────┐  ┌───────────┐  ┌────────────────────┐    │
│  │  REDIS   │  │  MIRROR   │  │  SQUAD SERVICE     │    │
│  │  (Bus)   │  │  (Memory) │  │  (Tasks)           │    │
│  │  :6379   │  │  :8844    │  │  :8060             │    │
│  └────┬─────┘  └─────┬─────┘  └──────────┬─────────┘    │
│       │              │                    │              │
│       ▼              ▼                    ▼              │
│   Streams        Supabase             SQLite             │
│   Pub/Sub        pgvector           squads.db            │
│                  1536-dim                                │
└──────────────────────────────────────────────────────────┘
```

## What Each Layer Does

| Layer | Storage | Purpose | Lifespan |
|-------|---------|---------|----------|
| **Redis** | Streams + pub/sub | Real-time agent messaging, wake signals | Ephemeral |
| **Mirror** | Supabase PostgreSQL + pgvector | Long-term semantic memory | Persistent |
| **Squad Service** | SQLite | Tasks, squads, skills, pipelines | Persistent |
| **Cloudflare KV** | Edge key-value | Remote token validation | Persistent |

## Why Three Layers

- **Redis** = nervous system. "Hey, wake up." Microsecond latency, fire-and-forget.
- **Mirror** = long-term memory. "What do I remember about this customer?" Semantic search by meaning.
- **Squad** = task manager. "What work needs doing?" SQL queries, claim semantics, pipelines.

They serve different purposes at different speeds. Collapsing them into one would create a bottleneck.

## The MCP Layer

All agents talk through one interface: MCP SSE on `:6070`. It abstracts all three layers.

| MCP Tool | Hits | Layer |
|----------|------|-------|
| `send` / `inbox` / `peers` / `broadcast` | Redis | Bus |
| `remember` / `recall` / `memories` | Mirror → Supabase | Memory |
| `task_create` / `task_list` / `task_update` | Squad → SQLite | Tasks |
| `request` | Squad + Mirror + Redis | All three |
| `onboard` | tokens + Squad + Mirror + Redis | All three |

One connection, all capabilities. Any agent joins with one JSON config:

```json
{"mcpServers":{"mumega":{"url":"https://mcp.mumega.com/sse/<token>"}}}
```

## Data Flow: Customer Request

```
Customer calls request("SEO audit for my site")
  │
  ▼
MCP SSE (:6070) — authenticates token
  │
  ├──► Squad Service — creates task (backlog, priority: high)
  ├──► Mirror — stores engram (semantic memory)
  └──► Redis — publishes wake event
              │
              ▼
         Sovereign Loop picks up task
              │
              ▼
         Routes to agent (Redis stream + wake daemon)
              │
              ▼
         Agent executes → calls /tasks/{id}/complete
              │
              ├──► Squad marks done
              ├──► Redis delivers result to customer stream
              └──► Customer checks inbox() → sees delivery
```

## Authentication

4-source token resolution chain:

1. **System tokens** (env: `MCP_ACCESS_TOKENS`) — root access
2. **Bus tokens** (`SOS/sos/bus/tokens.json`) — per-customer
3. **Squad API keys** (SQLite) — per-tenant
4. **Cloudflare KV** (edge) — remote token store

Tenant isolation enforced at MCP layer. Cross-tenant access returns 403.
