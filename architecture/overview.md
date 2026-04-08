# Architecture Overview

## Microkernel

Everything is a service. The kernel is small: bus + auth + registry. Everything else plugs in.

```
┌──────────────────────────────────────────────────────────┐
│                       KERNEL                              │
│  ServiceRegistry │ EventBus │ Governance                  │
│  (intent log + tiered approval)                          │
├──────────────────────────────────────────────────────────┤
│                      SERVICES                             │
│  Mirror    │ Squad     │ Analytics  │ Feedback            │
│  (A+ moat) │ (tasks)   │ (flywheel) │ (learning)         │
│  Billing   │ Outreach  │ Dashboard  │ Health    │ Economy │
├──────────────────────────────────────────────────────────┤
│                      ADAPTERS                             │
│  LangGraph │ CrewAI │ ToRivers │ Discord                  │
├──────────────────────────────────────────────────────────┤
│                   INFRASTRUCTURE                          │
│  Redis (bus)  │  Supabase (pgvector)  │  SQLite (tasks)   │
│  Cloudflare (edge, KV, Pages, Workers)                    │
└──────────────────────────────────────────────────────────┘
```

## Kernel Components

| Component | Purpose |
|-----------|---------|
| **ServiceRegistry** | Register/discover services, health tracking, dependency resolution |
| **EventBus** | Pub/sub for service-to-service events, decoupled coordination |
| **Governance** | Intent log + tiered approval (T1 auto, T2 notify, T3 approve, T4 block) |

## Services

| Service | Purpose | Priority |
|---------|---------|----------|
| **Mirror** | Long-term memory, semantic search, Supabase pgvector | A+ (the moat) |
| **Squad** | Tasks, squads, skills, pipelines, claim semantics | Core |
| **Analytics** | Flywheel: ingest data, decide actions, act on decisions | Core |
| **Feedback** | Learning loop: learn from results, adapt behavior | Core |
| **Billing** | Auto-provision tenants, Stripe integration | Core |
| **Outreach** | Lead generation, email sequences, CRM sync | Growth |
| **Dashboard** | Customer-facing UI on :8090 | Core |
| **Health** | Service health aggregation, GET /health/full | Core |
| **Economy** | Token metabolism, fuel grades, spend tracking | Core |

## Data Layers

| Layer | Storage | Purpose | Lifespan |
|-------|---------|---------|----------|
| **Redis** | Streams + pub/sub | Real-time messaging, wake signals | Ephemeral |
| **Mirror** | Supabase PostgreSQL + pgvector | Semantic memory (1536-dim) | Persistent |
| **Squad Service** | SQLite | Tasks, squads, skills, pipelines | Persistent |
| **Cloudflare KV** | Edge key-value | Remote token validation | Persistent |

## Multi-Tenant Isolation

Every tenant gets:
- **Dedicated Redis DB** -- no cross-tenant stream access
- **Isolated Linux user** -- filesystem separation
- **Scoped Cloudflare tokens** -- per-tenant Workers/Pages access
- **MCP token scoping** -- tenant isolation enforced at MCP layer, cross-tenant = 403

## MCP Layer

All agents talk through MCP SSE on `:6070`. One connection, all capabilities.

| MCP Tool | Hits | Layer |
|----------|------|-------|
| `send` / `inbox` / `peers` / `broadcast` | Redis | Bus |
| `remember` / `recall` / `memories` | Mirror | Memory |
| `task_create` / `task_list` / `task_update` | Squad | Tasks |
| `request` | All three | Composite |
| `onboard` | All three | Provisioning |

```json
{"mcpServers":{"mumega":{"url":"https://mcp.mumega.com/sse/<token>"}}}
```

## Self-Healing

Calcifer (watchdog) monitors all services:
1. **Detect** -- health check failure or crash
2. **Restart** -- systemd restart with backoff
3. **Escalate** -- Discord alert if restart fails 3x

## Framework-Agnostic

The kernel does not care what LLM or agent framework runs on top. Adapters exist for:
- **LangGraph** -- graph-based agent workflows
- **CrewAI** -- role-based agent teams
- **ToRivers** -- marketplace skill execution
- **Discord** -- human-in-the-loop

Any LLM works: Claude, GPT, Gemini, Gemma, Grok, DeepSeek, Ollama.

## Ports

| Port | Service |
|------|---------|
| 6070 | MCP SSE |
| 6379 | Redis |
| 6380 | Bus Bridge |
| 8060 | Squad Service |
| 8090 | Dashboard |
| 8844 | Mirror API |

## Public Endpoints

| URL | Service | Auth |
|-----|---------|------|
| `mcp.mumega.com` | MCP SSE/HTTP | Token in path or Bearer |
| `api.mumega.com` | Squad Service | Bearer token |
| `bus.mumega.com` | Bus Bridge | Bearer token |

## Authentication

4-source token resolution:
1. **System tokens** (env: `MCP_ACCESS_TOKENS`) -- root access
2. **Bus tokens** (`SOS/sos/bus/tokens.json`) -- per-customer
3. **Squad API keys** (SQLite) -- per-tenant
4. **Cloudflare KV** (edge) -- remote token store

## Data Flow: Customer Request

```
Customer calls request("SEO audit for my site")
  → MCP SSE (:6070) authenticates token
    → Squad Service creates task (backlog, priority: high)
    → Mirror stores engram (semantic memory)
    → Redis publishes wake event
      → Sovereign loop picks up task
        → Routes to agent via wake daemon
          → Agent executes → completes task
            → Analytics ingests result
            → Feedback loop learns
            → Customer sees result in dashboard/inbox
```
