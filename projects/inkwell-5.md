# Inkwell 5

**Version:** v5.0.0-alpha.1
**Updated:** 2026-04-15
**Deployed:** `inkwell-api.weathered-scene-2272.workers.dev`

---

## What It Is

Inkwell is an Astro-based content framework and publishing engine. Config-driven, agent-first, and designed to be forked. v5 ships the fork-ready architecture with multi-tenant support and a production Cloudflare Worker API.

## Stack

- **Frontend:** Astro 5, React 19, TailwindCSS 4
- **Worker:** Hono on Cloudflare Workers
- **Storage:** D1 (analytics, core, marketing), KV (content), R2 (media)
- **Search:** Pagefind (static)
- **Tests:** Vitest with Cloudflare Workers pool

## Route Groups (14)

Pages are organized as route groups under `src/pages/`:

| Group | Routes |
|-------|--------|
| `blog/` | Post listing + individual posts |
| `portal/` | Authenticated area (Telegram approval, user dashboard) |
| `products/` | Product pages |
| `tools/` | AI tools pages |
| `labs/` | Experimental features |
| `topics/` | Topic index pages |
| `tag/` | Tag filtered views |
| `team/` | Author/agent profiles |
| Root pages | index, about, explore, search, pricing, services, vision, subscribe, quiz, ai-readiness, iso-42001, starter-kit, studio, whitepaper, terms, privacy, 404 |

## Inkwell Worker API

Deployed Cloudflare Worker at `inkwell-api.weathered-scene-2272.workers.dev`.

### Core Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Health check |
| `POST /api/publish` | Publish content (auth: Bearer token) |
| `GET /api/posts` | List published posts |
| `GET /api/posts/:slug` | Get post markdown + meta |
| `GET /api/drafts` | List drafts (auth required) |
| `POST /api/view` | Record page view (analytics) |
| `POST /api/reaction` | Record emoji reaction |
| `GET /api/reactions/:slug` | Get reaction counts |
| `POST /api/subscribe` | Email subscribe |
| `POST /api/unsubscribe` | Email unsubscribe |
| `GET /api/stats/:slug` | Views + reactions for slug |

### Additional Route Groups (Worker)
- `/api/auth` — Magic link + OTP auth (email/phone)
- `/api/payments` — Glass Commerce (Stripe payment links)
- `/api/telegram` — Telegram bot webhook (approval commands)
- `/mcp` — MCP tool endpoints

### D1 Databases
- `inkwell-analytics` — page_views, reactions, subscribers, content_index
- `inkwell-core` — core application data
- `inkwell-marketing` — marketing / lead data

## Publish Flow

`POST /api/publish` with Bearer token:
1. Validates payload (title, content, optional slug/tags/description/status)
2. Slug uniqueness check against KV + D1
3. Writes markdown to `CONTENT` KV: `post:{slug}`
4. Writes metadata to KV: `meta:{slug}`
5. Indexes in D1 `content_index`
6. Triggers Cloudflare Pages deploy hook if configured

## Telegram Approval Commands

Agents send content to a Telegram draft queue. Humans approve via bot commands in the portal:
- `/approve {draftId}` — publish the draft
- `/reject {draftId}` — discard
- Handled by `workers/inkwell-api/src/routes/telegram.ts`

## Glass Commerce

`/api/payments` — Stripe payment link generation:
- Create one-time or recurring payment links
- Returns checkout URL for sharing with clients
- Exposed to customers via the `sell` MCP tool

## Fork-Ready Design

Zero hardcoded references. All configuration via `inkwell.config.ts`.

Three docs included:
- `FORK-GUIDE.md` — step-by-step fork + deploy
- `CONFIG-REFERENCE.md` — every configuration option
- `API-REFERENCE.md` — Worker API reference

## Key Configuration

`inkwell.config.ts` controls:
- Theme colors (→ CSS custom properties)
- Feature flags (ReadingProgress, Reactions, ShareButtons, NewsletterCTA)
- Analytics (GA, GTM, Clarity, Hotjar, Plausible)
- i18n (locale, RTL support, hreflang)
- SEO (site name, description, JSON-LD)

## Commands

```bash
npm run dev          # Dev server
npm run build        # Production build
npm run deploy       # Build + deploy to Cloudflare Pages
npm run ingest       # Process content/inbox/ → content/en/
npm run generate:og  # Generate OG images (Playwright)
npm run flywheel     # Run content flywheel (HN RSS scorer)
```
