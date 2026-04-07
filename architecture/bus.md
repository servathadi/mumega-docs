# Bus (Redis)

The nervous system. Real-time messaging between agents.

## How It Works

```
Agent A sends message
  → Redis stream (durable)
  → Wake daemon catches it (pub/sub)
  → tmux send-keys injection (local agents)
  → OR OpenClaw wake channel (remote agents)
  → Agent B sees it as user input
  → Agent B responds
```

## Streams

| Stream Pattern | Purpose |
|---------------|---------|
| `sos:stream:global:agent:{name}` | Global agent inbox |
| `sos:stream:project:{project}:agent:{name}` | Project-scoped inbox |
| `sos:stream:sos:channel:private:agent:{name}` | Legacy inbox (wake daemon reads this) |
| `sos:stream:sos:channel:global` | Global broadcast |

## Wake Daemon

`scripts/agent-wake-daemon.py` — systemd service, subscribes to `sos:wake:{agent}` pub/sub.

When a message arrives:
1. Check if agent's tmux session exists
2. Check if agent is at a prompt (not mid-response)
3. If ready: `tmux send-keys -t {session} "[bus:{agent}] {message}" Enter`
4. If busy: message stays in stream for later

### Agent Routing

```python
TMUX_AGENTS = {"kasra", "mumega", "codex", "spai"}   # tmux send-keys
OPENCLAW_AGENTS = {"athena", "worker", "sol", "dandan", "gemma", "river"}  # pub/sub wake
```

## Bus Bridge

`SOS/sos/bus/bridge.py` — HTTP proxy for remote agents (`:6380`, public at `bus.mumega.com`).

Remote agents (Cyrus on Mac, Antigravity) can't reach Redis directly. The bridge accepts HTTP requests with bearer token auth and proxies to Redis.

## Bus Tokens

`SOS/sos/bus/tokens.json` — per-customer scoped tokens.

Each token has: token string, project scope, label, active flag, created_at.

## MCP Tools

- `send(to, text)` — send message to specific agent
- `inbox(agent, limit)` — check messages
- `peers()` — list online agents
- `broadcast(text)` — message all agents
- `ask(agent, text)` — send and wait for reply
