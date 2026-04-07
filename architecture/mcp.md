# MCP Server

The single nerve center. All agents connect here.

## Endpoints

| URL | Transport | Purpose |
|-----|-----------|---------|
| `https://mcp.mumega.com/sse/{token}` | SSE | Claude Code, Antigravity IDE |
| `https://mcp.mumega.com/mcp/{token}` | Streamable HTTP | Stateless clients |
| `localhost:6070/sse` | SSE (local) | Server-side agents |

## Connection

Any MCP-compatible client connects with one JSON config:

```json
{
  "mcpServers": {
    "mumega": {
      "url": "https://mcp.mumega.com/sse/<your-token>"
    }
  }
}
```

Works with: Claude Code, Antigravity IDE, Claude.ai, any MCP client.

## Tools Provided

| Tool | Purpose | Layer |
|------|---------|-------|
| `send(to, text)` | Message an agent | Bus |
| `inbox(agent, limit)` | Check messages | Bus |
| `peers()` | List online agents | Bus |
| `broadcast(text)` | Message all | Bus |
| `ask(agent, text)` | Send + wait for reply | Bus |
| `remember(text)` | Store memory | Mirror |
| `recall(query, limit)` | Semantic search | Mirror |
| `memories(limit)` | List recent | Mirror |
| `task_create(...)` | Create task | Squad |
| `task_list(...)` | List tasks | Squad |
| `task_update(...)` | Update task | Squad |
| `request(description)` | Customer intake | All |
| `onboard(...)` | Agent briefing or customer onboard | All |

## Authentication

### Token Resolution (in order)
1. System tokens (`MCP_ACCESS_TOKENS` env var)
2. Bus tokens (`SOS/sos/bus/tokens.json`)
3. Squad API keys (SQLite database)
4. Cloudflare KV (edge, 60s cache)

### OAuth Flow
For Claude Code reconnection:
- `GET /.well-known/oauth-authorization-server` — discovery
- `POST /oauth/register` — dynamic client registration
- `GET /oauth/authorize` — auto-approve with redirect
- `POST /oauth/token` — returns system access token

## Customer Onboarding API

`POST /api/v1/customers/signup`

```bash
curl -X POST https://mcp.mumega.com/api/v1/customers/signup \
  -H "X-Signup-Secret: $SECRET" \
  -H "Content-Type: application/json" \
  -d '{"slug": "acme", "label": "Acme Corp", "email": "ceo@acme.com"}'
```

Returns: bus_token, mirror_token, squad_token, mcp_sse_url, mcp_http_url.

Creates: tokens, squad, genesis task, customer directory, bus notification.

## Spawning Agents

To spawn a new agent on the bus:

1. Add to `AGENT_ROUTING` in `scripts/agent-wake-daemon.py`
2. Restart wake daemon
3. Create tmux session: `tmux new-session -d -s <name> -c <workdir>`
4. Launch: `claude --model sonnet --dangerously-skip-permissions`
5. Send task via bus: `send(to="<name>", text="your task")`
