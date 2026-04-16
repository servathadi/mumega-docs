# MCP Product Architecture

**Updated:** 2026-04-15

---

## What It Is

Mumega is an MCP server that gives any AI a business operating system. One config line connects Claude Code, Claude Desktop, Cursor, ChatGPT, or any MCP-compatible client to memory, tasks, content publishing, payments, and a marketplace of agent squads.

## Connection

```json
{
  "mcpServers": {
    "mumega": {
      "url": "https://mcp.mumega.com/sse/<your-token>"
    }
  }
}
```

Claude Code: `claude mcp add mumega --transport sse --url "https://mcp.mumega.com/sse/<token>"`

## Customer Tools (13)

### Core (8)

| Tool | Purpose |
|------|---------|
| `remember(text, context?)` | Save to business memory — persists across all conversations |
| `recall(query, limit?)` | Semantic search over memory |
| `publish(title, content, slug?, status?, tags?)` | Publish content to Inkwell-powered site |
| `dashboard(period?)` | Website traffic, leads, revenue metrics |
| `create_task(title, description?, priority?, labels?)` | Delegate work to AI squad |
| `list_tasks(status?)` | See squad work queue |
| `sell(product_name, price_cents, currency?, description?)` | Generate Stripe checkout link |
| `my_site()` | Site URL, status, recent posts, active features |

### Marketplace (5)

| Tool | Purpose |
|------|---------|
| `browse_marketplace(query?, category?)` | Browse squads and tools for hire |
| `subscribe(listing_id)` | Subscribe to a marketplace listing |
| `my_subscriptions()` | View active subscriptions |
| `create_listing(title, description, category, listing_type, price_cents, tags?)` | List your squad/tool for sale |
| `my_earnings()` | MRR breakdown for your listings |

## Token Scoping

- **Customer tokens** — 13 tools above. No bus access, no cross-tenant visibility, no admin tools.
- **System tokens** — Full 16+ tools including bus (`send`, `inbox`, `peers`, `broadcast`, `ask`), admin (`onboard`, `status`, `search_code`, `task_board`), and raw memory tools.
- Scope is enforced at the MCP layer. Blocked tools return `tool_not_found`.

## Architecture

```
Customer AI (Claude / Cursor / ChatGPT)
    │
    │  SSE or Streamable HTTP
    ▼
mcp.mumega.com  (:6070 locally)
    │
    ├── Mirror (:8844)      ← remember / recall
    ├── Squad Service (:8060) ← create_task / list_tasks
    ├── Inkwell Worker       ← publish
    ├── Glass Commerce       ← sell
    └── SaaS Service (:8075) ← dashboard / my_site / marketplace
```

## Token Resolution Order

1. System tokens (`MCP_ACCESS_TOKENS` env var)
2. Bus tokens (`SOS/sos/bus/tokens.json`) — hot-reloaded every 30s via mtime check
3. Squad API keys (SQLite in squads.db)
4. Cloudflare KV (edge cache, 60s TTL)

## Multi-Seat Support

Plans include seat limits. Each seat is a separate MCP token scoped to the same tenant.

| Plan | Seats |
|------|-------|
| Starter $29 | 1 |
| Growth $79 | 5 |
| Scale $199 | Unlimited |

Generate a seat: `POST /tenants/{slug}/seats` on SaaS Service.

## Distribution Targets

- Claude Code (MCP native)
- Claude Desktop
- Cursor
- ChatGPT (via action URL)
- OpenClaw / Antigravity IDE
- Any MCP-compatible client
