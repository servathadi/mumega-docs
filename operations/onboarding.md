# Customer Onboarding

One API call creates a fully functional customer instance.

## API Endpoint

```bash
POST https://mcp.mumega.com/api/v1/customers/signup
```

Headers:
- `X-Signup-Secret: <secret>` OR
- `Authorization: Bearer <system-token>`

Body:
```json
{
  "slug": "acme-corp",
  "label": "Acme Corporation",
  "email": "ceo@acme.com"
}
```

Response:
```json
{
  "status": "ok",
  "slug": "acme-corp",
  "bus_token": "sk-bus-acme-corp-...",
  "mirror_token": "sk-mumega-acme-corp-...",
  "squad_token": "sk-squad-acme-corp-...",
  "mcp_sse_url": "https://mcp.mumega.com/sse/sk-bus-acme-corp-...",
  "mcp_http_url": "https://mcp.mumega.com/mcp/sk-bus-acme-corp-..."
}
```

## What Gets Created

1. **Mirror tenant key** — scoped memory access
2. **Bus token** — scoped messaging
3. **Squad API key** — scoped task management
4. **Default squad** — `{slug}-dev` with project objective
5. **Genesis task** — "Welcome — initial audit" in backlog
6. **Customer directory** — `~/.mumega/customers/{slug}/`
   - `CLAUDE.md` — project context
   - `.claude/settings.json` — MCP auto-config
   - `.env` — secrets (gitignored)
   - `README.md` — setup instructions
7. **Mirror engram** — onboarding record
8. **Bus notification** — team alerted

## MCP Tool

System agents can onboard customers through the bus:

```
mcp__sos__onboard(mode="customer", slug="acme-corp", label="Acme Corporation", email="ceo@acme.com")
```

## Customer Connects

After onboarding, customer connects any MCP client:

```json
{"mcpServers":{"mumega":{"url":"https://mcp.mumega.com/sse/<bus_token>"}}}
```

Then requests work:
```
request(description="SEO audit for my dental site", priority="high")
```

Checks results:
```
inbox()
```

## Tenant Isolation

- Customer can only see their own memories, tasks, and messages
- Cross-tenant access returns 403
- System tokens bypass isolation (internal agents only)
