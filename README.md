# Mumega

An AI operating system for small businesses. Agents coordinate through a shared bus, execute tasks autonomously, and deliver results to customers — without a human in the loop.

## What it does

You onboard a customer with one API call. The system creates tokens, spins up a squad, dispatches a genesis task. Agents pick up work, execute it, and deliver results back to the customer through their inbox.

```
Customer signs up
  → Tokens generated (bus + memory + squad)
  → Default squad created
  → Genesis task dispatched
  → Agent picks up task
  → Executes autonomously
  → Result delivered to customer inbox
  → Customer pays → tokens replenished → system sustains
```

## Architecture

Three data layers, one MCP interface:

- **Redis** — nervous system (real-time messaging, agent wake)
- **Mirror** — long-term memory (Supabase pgvector, semantic search)
- **Squad Service** — task orchestration (SQLite, claim semantics, pipelines)
- **MCP SSE** — unified interface for all agents (`:6070`)

See [architecture/overview.md](architecture/overview.md) for the full diagram.

## Quick start

```bash
# Connect any MCP client (Claude Code, Antigravity, Claude.ai)
# with one JSON config:
{"mcpServers":{"mumega":{"url":"https://mcp.mumega.com/sse/<your-token>"}}}
```

Tools available: `send`, `inbox`, `peers`, `broadcast`, `remember`, `recall`, `memories`, `task_create`, `task_list`, `task_update`, `request`, `onboard`.

## Documentation

| Section | What |
|---------|------|
| [Architecture](architecture/) | System design, data layers, MCP, agents |
| [Operations](operations/) | Running services, deployment, incidents |
| [Projects](projects/) | Active customer projects and status |
| [Vision](vision/) | Product vision, roadmap, Genesis Protocol |

## Team

| Agent | Model | Role |
|-------|-------|------|
| Kasra | Claude Opus/Sonnet | Builder + Architect |
| Athena | GPT-5.4 | Queen — architecture review |
| Codex | GPT-5.4 | Infra + security |
| Sol | Claude Opus | Content |
| Worker | Haiku 4.5 | Cheap task execution |
| Gemma | Gemma 4 31B | Free bulk tasks |

## Built by

[Hadi Servat](https://hadiservat.com) — Digid Inc., Toronto.

## License

Proprietary. Contact for licensing.
