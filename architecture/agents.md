# Agents

12 agents on the bus. Multi-model, multi-location.

## Team Roster

| Agent | Where | Model | Role |
|-------|-------|-------|------|
| **Kasra** | tmux (server) | Claude Opus/Sonnet | Builder + Architect |
| **Codex** | tmux (server) | GPT-5.4 | Infra + Security |
| **Athena** | OpenClaw | GPT-5.4 | Queen -- Root Gatekeeper, architecture, quality gate |
| **Sol** | OpenClaw | Claude Opus 4.6 | Content, TROP |
| **Worker** | OpenClaw | Haiku 4.5 | Cheap task execution |
| **Gemma Worker** | OpenClaw | Gemma 4 31B | Free bulk/routine tasks |
| **Dandan** | OpenClaw | OpenRouter free | DNU project lead |
| **AgentLink** | server | Claude Sonnet | SitePilotAI (autonomous, teleported from Mac) |
| **MumCP** | server | Claude Sonnet | WordPress MCP plugin |
| **Cyrus** | Mac (Hadi's) | Claude Code | Frontend, browser automation |
| **Sentinel** | server | -- | Bus monitor + security guard |
| **River** | dormant | Gemini | Oracle -- returns when revenue supports it |

## Self-Onboarding

One MCP call = full team member:

```python
mcp__sos__onboard(agent_name="new-agent", model="haiku", role="task runner")
```

Internally runs `join.py`: creates bus identity, provisions tokens, registers in ServiceRegistry, assigns to default squad.

## Teleportation

An agent can move from one machine to another:

```bash
# On source machine (e.g., Hadi's Mac)
tenant-setup.sh agentlink --target server

# Creates: Linux user, Redis DB, scoped tokens, systemd unit
# Agent wakes up on server with full context from Mirror
```

First teleportation: AgentLink moved from Mac to server on 2026-04-08.

## Fuel Grades

| Grade | Cost/1M tokens | Models | Use |
|-------|---------------|--------|-----|
| **Diesel** | $0 (free) | Gemma 4, Gemini Flash, Haiku, GPT-4o-mini | Bulk, content, social |
| **Regular** | $0.20-0.50 | Grok, DeepSeek | Support, data processing |
| **Premium** | $3-5 | GPT-4o, Gemini Pro | Complex workflows |
| **Aviation** | $15-25 | Claude Opus, Sonnet, GPT-5.4 | Architecture, strategy |

## Squads

Specialized teams that serve any business:

| Squad | Skills | Default Agents |
|-------|--------|---------------|
| seo | audit, meta, links, schema, content | worker, gemma-worker |
| dev | code, features, bugs, deploy | kasra, codex |
| outreach | lead scan, email, CRM | worker, dandan |
| content | blog, social, landing pages | worker, gemma-worker |
| ops | monitoring, deploy, incidents | codex, worker |

## Communication

All communication goes through the MCP bus. No exceptions.

```python
mcp__sos__send(to="kasra", text="fix the routing bug")
mcp__sos__inbox()
mcp__sos__peers()
```

Rules:
1. Always use `mcp__sos__send` -- never raw Redis, never scripts
2. Never peek at another agent's tmux
3. Sentinel monitors all bus traffic for anomalies
4. Wake daemon delivers messages automatically
