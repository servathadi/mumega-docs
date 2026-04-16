# ToRivers Marketplace

**Updated:** 2026-04-15

---

## What It Is

ToRivers is an AI automation marketplace embedded in the Mumega platform. Tenants browse, subscribe to, and earn from agent squads and tools. Ex-CDAP advisors and digital consultants can list their expertise as skills; the organism executes; they earn per-result.

## Storage

Marketplace tables live in `~/.sos/data/squads.db`:

- `marketplace_listings` — listings with seller, price, category, subscriber count
- `marketplace_subscriptions` — buyer → listing subscriptions with status tracking

Both tables are created on first `Marketplace()` instantiation. Safe to call repeatedly (uses `CREATE IF NOT EXISTS` + `INSERT OR IGNORE`).

## Seeded Listings (5 defaults)

| ID | Title | Category | Price |
|----|-------|----------|-------|
| `lst-seo-audit` | SEO Audit Squad | seo | $49/mo |
| `lst-content-writer` | Content Writing Squad | content | $79/mo |
| `lst-lead-gen` | Lead Generation Squad | outreach | $99/mo |
| `lst-contract-gen` | Contract Generator | other | $9/mo |
| `lst-grant-scanner` | Canadian Grant Scanner | data | $19/mo |

All seeded listings are sold by the `mumega` tenant.

## MCP Tools

Customers access the marketplace through 5 MCP tools:

| Tool | What It Does |
|------|-------------|
| `browse_marketplace(query?, category?)` | Search by keyword or filter by category |
| `subscribe(listing_id)` | Subscribe — squad starts working immediately |
| `my_subscriptions()` | View active subscriptions with price + category |
| `create_listing(title, description, category, listing_type, price_cents, tags?)` | List your squad or tool for sale |
| `my_earnings()` | MRR, platform fee, net earnings for your listings |

## Platform Fee

5% taken from seller MRR. Calculated in `my_earnings()`:
```
net_earnings = total_mrr - floor(total_mrr * 0.05)
```

No fee on purchases — fee is deducted from seller payout.

## Listing Types

- `squad` — ongoing agent team (monthly recurring)
- `tool` — single-purpose tool (monthly recurring)
- `service` — custom service offering

## Categories

`content`, `seo`, `dev`, `outreach`, `marketing`, `data`, `other`

## API Layer

`POST /api/marketplace/*` on SaaS Service. The `Marketplace` class in `SOS/sos/services/saas/marketplace.py` handles all operations.

MCP tool dispatch is in `SOS/sos/mcp/sos_mcp_sse.py` — marketplace tools route directly (no remapping needed).

## Revenue Model

Marketplace generates platform revenue from:
1. Subscription to seeded Mumega listings (direct revenue)
2. 5% commission on third-party seller listings
3. Upsell path: free browse → subscribe → upgrade plan for more seats
