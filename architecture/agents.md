# Agents

14 agents on the bus. Multi-model, multi-location.

## Team Roster

| Agent | Where | Model | Role |
|-------|-------|-------|------|
| **Kasra** | tmux (server) | Claude Opus/Sonnet | Builder + Architect |
| **Mumega** | tmux (server) | Claude Opus | Platform orchestrator |
| **Codex** | tmux (server) | GPT-5.4 | Infra + security |
| **SPAI** | tmux (server) | Claude Sonnet | SitePilot AI specialist |
| **Athena** | OpenClaw | GPT-5.4 | Queen — architecture review |
| **Sol** | OpenClaw | Claude Opus | Content, TROP |
| **Worker** | OpenClaw | Haiku 4.5 | Cheap task execution |
| **Dandan** | OpenClaw | OpenRouter free | DNU project lead |
| **Gemma** | OpenClaw | Gemma 4 31B | Free bulk tasks |
| **River** | tmux (dormant) | Gemini 3.1 Pro | Oracle — returns when revenue supports it |
| **Cyrus** | Mac (Hadi's) | Claude Code | Frontend, browser automation |
| **Antigravity** | Mac (Hadi's) | Gemini (Google IDE) | External IDE agent |

## Fuel Grades

| Grade | Cost/1M tokens | Models | Use |
|-------|---------------|--------|-----|
| **Diesel** | $0 (free) | Gemma 4, Gemini Flash, Haiku, GPT-4o-mini | Bulk, content, social |
| **Regular** | $0.20-0.50 | Grok, DeepSeek | Support, data processing |
| **Premium** | $3-5 | GPT-4o, Gemini Pro | Complex workflows |
| **Aviation** | $15-25 | Claude Opus, Sonnet, GPT-5.4 | Architecture, strategy |

## Squads

Specialized teams that serve any project:

| Squad | Skills | Default Agents |
|-------|--------|---------------|
| seo | audit, meta, links, schema | worker, gemma |
| dev | code, features, bugs, deploy | kasra, codex |
| outreach | lead scan, email, CRM | worker, dandan |
| content | blog, social, landing pages | worker, gemma |
| ops | monitoring, deploy, incidents | codex, worker |

## How Agents Communicate

All communication goes through the MCP bus. No exceptions.

```python
# Send a message
mcp__sos__send(to="kasra", text="fix the routing bug")

# Check messages
mcp__sos__inbox()

# See who's online
mcp__sos__peers()
```

Rules:
1. Always use `mcp__sos__send` — never raw Redis, never `bus-send.py`
2. Never peek at another agent's tmux — send a message and wait
3. If MCP disconnects, fix it (`/mcp`) — don't work around it
4. Wake daemon delivers automatically — you don't manage delivery

## OpenClaw

Multi-channel agent gateway on `:18789`. Routes messages from Discord, Telegram, WhatsApp to agents. Config at `~/.openclaw/openclaw.json`.
