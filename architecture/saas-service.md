# SaaS Service

**Port:** 8075
**systemd unit:** `sos-saas`
**Updated:** 2026-04-15

---

## What It Does

Tenant lifecycle management for Mumega SaaS. Handles signup, provisioning, billing, multi-seat tokens, Inkwell build orchestration, and custom domain routing.

## Pricing Tiers

| Plan | Price | Seats | Description |
|------|-------|-------|-------------|
| Starter | $29/mo | 1 | 1 AI seat, basic memory + tasks |
| Growth | $79/mo | 5 | 5 seats, content publishing, Glass Commerce |
| Scale | $199/mo | Unlimited | Full platform + marketplace access |

## Tenant Registry

SQLite in `~/.sos/data/squads.db`. Each tenant record contains:

- `slug`, `label`, `email` — identity
- `subdomain` — `{slug}.mumega.com`
- `domain` — optional custom domain
- `plan`, `status` — `provisioning | active | suspended | cancelled`
- `bus_token` — MCP connection token
- `squad_id`, `mirror_project` — resource bindings
- `stripe_customer_id`, `stripe_subscription_id` — billing
- `inkwell_config` — per-tenant Inkwell overrides
- `telegram_chat_id` — for delivery notifications

## Self-Serve Signup

Landing page at `http://localhost:8075/` (served as HTML).

`POST /signup` — minimal signup, returns MCP config in under 10 seconds:
1. Generates `sk-{slug}-{hex}` bus token
2. Registers token in `tokens.json`
3. Creates tenant record
4. Activates tenant
5. Optionally creates Stripe customer + checkout session
6. Returns MCP connection configs for Claude Code, Claude Desktop, Cursor, ChatGPT

## Full Onboarding

`POST /onboard` — questionnaire to live site:
1. Creates tenant + squad in squads.db
2. Generates starter pages from questionnaire answers (home, about, services)
3. Triggers async Inkwell build
4. Sends Telegram notification when live

Questionnaire fields: `email`, `business_name`, `industry`, `services[]`, `primary_color`, `tagline`, `domain`, `plan`

## API Endpoints

### Tenant CRUD
| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/` | Self-serve signup page |
| `POST` | `/signup` | Quick signup → MCP config |
| `POST` | `/onboard` | Full questionnaire → live site |
| `POST` | `/tenants` | Create tenant (programmatic) |
| `GET` | `/tenants` | List all tenants |
| `GET` | `/tenants/{slug}` | Get tenant |
| `PUT` | `/tenants/{slug}` | Update tenant |
| `POST` | `/tenants/{slug}/activate` | Activate with squad + bus token |
| `POST` | `/tenants/{slug}/suspend` | Suspend tenant |

### Seats
| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/tenants/{slug}/seats` | Create new MCP seat |
| `GET` | `/tenants/{slug}/seats` | List seats |
| `DELETE` | `/tenants/{slug}/seats/{token_id}` | Revoke seat |

### Billing
| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/tenants/{slug}/usage` | Record usage metric |
| `GET` | `/tenants/{slug}/usage` | Get usage by period |
| `POST` | `/tenants/{slug}/transaction` | Record transaction |
| `GET` | `/tenants/{slug}/invoice` | Get tenant invoice |
| `GET` | `/revenue` | Platform revenue summary |

### Builds
| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/builds/enqueue/{slug}` | Queue an Inkwell build |
| `GET` | `/builds/status` | Build queue status |

### Domains
| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/tenants/{slug}/domain` | Set custom domain (provisions Cloudflare) |
| `DELETE` | `/tenants/{slug}/domain` | Remove custom domain |
| `GET` | `/resolve/{hostname}` | Resolve hostname → tenant (used by Inkwell Worker) |

## Build Orchestrator

When `POST /builds/enqueue/{slug}` is called:
1. `BuildQueue` enqueues the build with priority
2. `builder.py` runs an Astro build in the tenant context
3. Built assets are uploaded to Cloudflare KV
4. Telegram notification sent to `tenant.telegram_chat_id` on success/failure

## Stripe Integration

Set env vars to enable:
- `STRIPE_SECRET_KEY`
- `STRIPE_PRICE_STARTER`, `STRIPE_PRICE_GROWTH`, `STRIPE_PRICE_SCALE`

Signup creates a Stripe Customer and (for paid plans) a Checkout Session. Checkout URL returned in signup response.
