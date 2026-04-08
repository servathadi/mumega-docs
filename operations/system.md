# System -- What's Running

## Services (systemd user units)

| Service | Port | Restart | Purpose |
|---------|------|---------|---------|
| `sos-mcp-sse` | 6070 | always | MCP SSE/HTTP server |
| `sos-squad` | 8060 | always | Squad Service -- tasks, skills, pipelines |
| `mirror` | 8844 | always | Memory API (Supabase pgvector) |
| `dashboard` | 8090 | always | Customer-facing dashboard |
| `calcifer` | -- | always | Watchdog + self-healing (detect, restart, escalate) |
| `sentinel` | -- | always | Bus security monitor |
| `agent-wake-daemon` | -- | always | Redis pub/sub -> tmux injection |
| `bus-bridge` | 6380 | on-failure | HTTP proxy for remote agents |

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

## Flywheel Timers (Monday cycle)

| Time | Step | What |
|------|------|------|
| 5:30 AM | Feedback | Collect results from past week, score outcomes |
| 6:00 AM | Ingest | Pull analytics data (traffic, conversions, rankings) |
| 7:00 AM | Decide | Brain scores portfolio, prioritizes next actions |
| 8:00 AM | Act | Dispatch tasks to squads based on decisions |

## Health Check

```bash
# Full system health
curl https://mcp.mumega.com/health/full

# Individual service
systemctl --user status sos-mcp-sse
systemctl --user status sos-squad
systemctl --user status mirror
systemctl --user status dashboard
systemctl --user status calcifer
systemctl --user status sentinel
```

## Docker Compose

All services available via docker-compose for local dev or new deployments:

```bash
docker-compose up -d        # start all
docker-compose logs -f      # tail logs
docker-compose down         # stop all
```

## Self-Healing (Calcifer)

1. Health check fails or service crashes
2. Calcifer detects within 30s
3. Restarts service via systemd
4. If restart fails 3x, escalates to Discord
5. Logs incident in Mirror for post-mortem
