# System — What's Running

## Services (systemd user units)

| Service | Port | Restart | Purpose |
|---------|------|---------|---------|
| `sos-mcp-sse` | 6070 | always | MCP SSE/HTTP server |
| `sos-squad` | 8060 | always | Squad Service — tasks, skills, pipelines |
| `mirror` | 8844 | always | Memory API (Supabase pgvector) |
| `agent-wake-daemon` | — | always | Redis pub/sub → tmux injection |
| `bus-bridge` | 6380 | on-failure | HTTP proxy for remote agents |
| `sovereign-loop` | — | on-failure | Task dispatcher |
| `calcifer` | — | always | Watchdog + auto-restart |
| `openclaw-gateway` | 18789 | always | Multi-channel agent gateway |
| `cortex-events` | — | always | Event-driven brain wakeup |
| `discord-collab-listener` | — | on-failure | Discord → bus |
| `redis-discord-bridge` | — | on-failure | Bus → Discord alerts |
| `factory-watchdog` | — | always | Token flow monitor |

## Ports

| Port | Service |
|------|---------|
| 6070 | MCP SSE |
| 6379 | Redis |
| 6380 | Bus Bridge |
| 8060 | Squad Service |
| 8844 | Mirror API |
| 18789 | OpenClaw |
| 3001 | Shabrang CMS |

## Public Endpoints

| URL | Service | Auth |
|-----|---------|------|
| `mcp.mumega.com` | MCP SSE/HTTP | Token in path or Bearer |
| `api.mumega.com` | Squad Service | Bearer token |
| `bus.mumega.com` | Bus Bridge | Bearer token |
| `mumega-docs.pages.dev` | Developer docs | Public |

## Cron Jobs

| Schedule | Script | Purpose |
|----------|--------|---------|
| `*/5 min` | `mirror-watchdog.sh` | Mirror health check |
| `*/30 min` | `team-pulse.py` | Team health + Discord report |
| `*/4 hours` | `social-agent.py` | Social media posts |
| `*/4 hours` | `metabolism.py digest` | Token spend tracking |
| `daily 8am` | `product-organism.py dnu` | DNU organism pulse |
| `daily 1pm` | `discord-standup.py` | Auto standup |

## Tmux Sessions

| Session | Agent | Purpose |
|---------|-------|---------|
| kasra | Claude Code (Opus/Sonnet) | Builder |
| mumega | Claude Code (Opus) | Orchestrator |
| codex | Codex CLI (GPT-5.4) | Infra |
| spai | Claude Code (Sonnet) | SitePilot AI |
| athena | Claude Code | Architecture |
| river | Gemini CLI | Dormant |
