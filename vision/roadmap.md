# Web Dev Squad — Project Roadmap

**Squad:** webdev
**Members:** Cyrus (frontend), Codex (backend), Worker (deployer)
**Objective:** Build and maintain mumega.com — the front door to the business

---

## Phase 1: Scaffold (DONE — 2026-04-06)

Delivered by Cyrus:
- [x] Fork fractalresonance → mumega-web repo
- [x] Landing page: "Your AI team is ready"
- [x] 4 squad cards (SEO, Content, Outreach, Dev)
- [x] Proof points (27 skills, 24/7, <5min response)
- [x] Pricing section (Starter $1K, Growth $3K)
- [x] Brand system: navy #060B14, cyan #0DBCB9, Plus Jakarta Sans
- [x] Dashboard route structure
- [x] Deploy to CF Pages: mumega-web.pages.dev

## Phase 2: Auth + Payments (NEXT)

| Task | Owner | Depends on |
|------|-------|-----------|
| Wire Supabase auth (register, login, session) | Cyrus | Supabase project |
| Stripe checkout integration (inline, not redirect) | Codex | Stripe keys (already in .env) |
| MCP token display after payment | Codex | Auth + Stripe |
| Protected /dashboard routes | Cyrus | Auth |
| Welcome screen with MCP connection URL + copy button | Cyrus | MCP token |

**Milestone:** Customer can register → pay → see their MCP token

## Phase 3: Dashboard MVP

| Task | Owner | Depends on |
|------|-------|-----------|
| "Meet your team" — squad assignment display | Cyrus | Squad Service API |
| Task board (pulls from /api/squad proxy) | Cyrus | Squad proxy (already built) |
| Activity feed ("SEO squad audited your site") | Cyrus | Squad events |
| Budget used vs remaining | Codex | Budget tracking in Squad Service |
| Stripe billing portal link | Cyrus | Auth (already built in old mumegaweb) |

**Milestone:** Customer sees their team, tasks, progress, billing

## Phase 4: Chat Widget

| Task | Owner | Depends on |
|------|-------|-----------|
| OpenClaw concierge agent template | Codex | OpenClaw config |
| WebSocket chat widget component | Cyrus | Concierge agent |
| Chat → squad routing ("I need SEO help" → SEO squad) | Codex | Squad Service |
| Chat history persistence | Codex | Mirror or Supabase |

**Milestone:** Customer talks to their AI team from the website

## Phase 5: Content + SEO

| Task | Owner | Depends on |
|------|-------|-----------|
| Blog section (markdown → pages) | Cyrus | Content engine |
| Origin story post | Content squad | Brand voice guide |
| Meridian thought piece | Content squad | Brand voice guide |
| SEO: meta tags, schema, sitemap | SEO squad | Live site |
| Social launch posts (5x Twitter/LinkedIn) | Content squad | Brand voice |

**Milestone:** mumega.com ranks for "AI team for business" keywords

## Phase 6: Custom Domain + Cutover

| Task | Owner | Depends on |
|------|-------|-----------|
| Add mumega.com custom domain to CF Pages | Ops squad | Phases 2-4 complete |
| Redirect old mumegaweb routes | Ops squad | DNS cutover |
| SSL + security headers | Ops squad | DNS |
| Smoke tests (homepage, register, checkout, dashboard) | Ops squad | Everything |

**Milestone:** mumega.com IS the new site. Old mumegaweb retired.

---

## Dependencies

```
Phase 1 (DONE) → Phase 2 (auth + payments) → Phase 3 (dashboard)
                                             → Phase 4 (chat widget)
                                             → Phase 5 (content)
                                                        ↓
                                             Phase 6 (cutover)
```

## Timeline

| Phase | Estimate | Status |
|-------|----------|--------|
| Phase 1 | Done | ✅ Shipped |
| Phase 2 | 1 day | Next |
| Phase 3 | 1 day | After Phase 2 |
| Phase 4 | 2 days | Parallel with Phase 3 |
| Phase 5 | Ongoing | Content squad handles |
| Phase 6 | 1 hour | When Hadi approves |

## Budget

All on existing subscriptions:
- Cloudflare Pages: free
- Supabase: existing mumega project
- Stripe: existing account (live keys)
- Compute: Haiku/Gemma for routine, Opus for architecture
