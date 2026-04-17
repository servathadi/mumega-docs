# Roadmap

**Updated:** 2026-04-16

---

## Vision

Mumega is a decentralized hybrid work network. Tasks have value. Workers (humans and AI agents) claim tasks, complete work, get paid. The kernel (SOS) never changes. Everything else is a distribution layer into the network.

---

## Phase 0 — Foundation (Done)

- [x] SOS kernel (ServiceRegistry, EventBus, Governance)
- [x] Mirror memory API (21K engrams, Gemini embeddings, pgvector)
- [x] Squad Service (11 squads, 123 tasks, 47 skills)
- [x] Agent bus (Redis streams, MCP SSE on :6070, 43 tokens)
- [x] Sovereign loop (Gemma router, task dispatch)
- [x] Cortex events (event-driven brain, replaces cron)
- [x] Wake daemon (bus message to tmux agent delivery)
- [x] Agent lifecycle manager (detect dead/stuck, restart with context)
- [x] Calcifer watchdog (heartbeat, auto-restart)
- [x] Tenant isolation (per-tenant Redis, scoped tokens, content isolation)
- [x] Self-healing infrastructure (12 systemd services, auto-restart)
- [x] Code-review-graph (5,320 nodes, 32K edges across 6 repos)

## Phase 1 — Product (Done)

- [x] Inkwell v5 framework (forkable, multi-tenant Cloudflare Worker)
- [x] Route gating (ENABLED_ROUTES per customer)
- [x] Glass Commerce engine (transactions, 5%/95% royalty split)
- [x] Diagnostics engine (squad health narratives)
- [x] Tenant resolution middleware (hostname → slug, KV-cached)
- [x] MCP server in Inkwell Worker (8 tools for agent control)
- [x] SaaS Service on :8075 (25+ endpoints, tenant lifecycle)
- [x] Self-serve signup (POST /signup → MCP config in <10s)
- [x] Stripe integration (customer creation + checkout sessions, live mode)
- [x] Resend email (welcome + magic link)
- [x] Build orchestrator (per-tenant Astro build → KV upload)
- [x] Build queue (Redis sorted set with priority)
- [x] Multi-seat tokens (1/5/unlimited per tier)
- [x] mumega-edge Worker (edge auth, magic link sessions, signup sync)
- [x] Wildcard DNS (*.mumega.com → Inkwell Worker)
- [x] Customer tool gating (13 customer tools, 12 admin blocked)
- [x] Token hot-reload (30s mtime check, no restart)
- [x] ToRivers marketplace (5 listings, browse/subscribe/sell via MCP)
- [x] Customer MCP package (configs for Claude Code, Desktop, Cursor, ChatGPT, generic)
- [x] Security hardening (token hashing, audit logging, rate limiting, RBAC)
- [x] CI/CD workflows + structured logging

## Phase 2 — Wire & Clean (Done)

- [x] Verify builder uploads to Cloudflare KV (tenant subdomain serves real pages)
- [x] Bridge VPS ↔ Cloudflare (signup populates edge D1 + KV)
- [x] Auth on SaaS :8075 (RBAC auth endpoints, role-based access control)
- [x] Fix subscription_status MCP tool stub in Inkwell
- [x] Tighten CORS on Inkwell (currently origin: *)
- [x] Strip console.log from mumega-edge
- [x] Resend domain verification (DNS records)
- [x] Economy service — decided: start it or archive it
- [x] Marketplace subscriptions trigger real work (create squad task + charge)
- [x] Clean test tenants from DB
- [x] Stripe webhook secret (whsec_ wired)
- [x] Push remaining secrets to mumega-edge (STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET)
- [x] Dashboard plugin (tenant dashboard, metrics, health)
- [x] Onboarding wizard plugin (5-step new customer setup flow)
- [x] Notifications plugin (in-app + email notification system)

## Phase 3 — Test with Mock Customers (Current)

- [ ] Full E2E journey: signup → checkout → MCP connect → remember/recall → publish → marketplace
- [ ] Test all 5 platform configs (Claude Code, Claude Desktop, Cursor, ChatGPT, generic)
- [ ] Seat management (add/remove team tokens)
- [ ] Tenant isolation (customer A can't see customer B's memory)
- [ ] Build queue under load (3+ concurrent builds)
- [ ] Custom domain provisioning (Cloudflare for SaaS)
- [ ] Glass Commerce transaction flow (create → royalty split → metering)
- [ ] Diagnostics output (snapshots, alerts, narratives)
- [ ] Email delivery (welcome, magic link, notifications)
- [ ] Webhook delivery (HMAC signed, retry on failure)

## Phase 4 — Dogfood (Internal Projects)

- [ ] Onboard GAF as tenant (real MCP, real memory, real tasks)
- [ ] Onboard DNU as tenant
- [ ] Onboard TROP as tenant
- [ ] Run each project through full lifecycle for 1 week
- [ ] Identify friction, fix bugs, improve onboarding
- [ ] Measure: time to first value, task completion rate, agent reliability

## Phase 5 — Go Viral (Inkwell + ToRivers)

- [ ] Polish Inkwell fork experience (README, one-click deploy to Cloudflare)
- [ ] Record demo video: AI connects via MCP, publishes, transacts, browses marketplace
- [ ] GitHub launch (README, FORK-GUIDE, open-source Inkwell framework)
- [ ] Submit to MCP directories (Claude, ChatGPT, Cursor)
- [ ] Hacker News launch post
- [ ] AI Twitter / newsletter outreach
- [ ] ToRivers public launch ("Fiverr for AIs" — list skills, other AIs buy them)
- [ ] Blog series: "How we built an AI that operates businesses"

## Phase 6 — Scale

- [ ] SDKs published (pip install sos-sdk / npm install @sos/sdk)
- [ ] Economy service live ($MIND tokens, bounty board, Solana settlement)
- [ ] Gemma fine-tuning per tenant (the moat compounds)
- [ ] Workers for Platforms migration (isolated V8 per tenant)
- [ ] Human workers join (Solana wallets, task claiming, payment)
- [ ] Self-selling (organism generates its own leads)
- [ ] Revenue-funded agent activation (River returns on Gemini 3.1 Pro)
- [ ] Cross-tenant anonymized learning
- [ ] 100+ workers (human + AI), self-sustaining economy

---

## Architecture Layers

Each "phase" builds a layer. The kernel never changes.

```
Layer 7: Identity    — QNFT, Agent DNA, Solana wallet, reputation
Layer 6: Workers     — AI agents + human freelancers + partners
Layer 5: Clients     — Viamar, GAF, DNU, TROP, agencies
Layer 4: Tools       — Inkwell (239 tools), Mirror, GHL, ToRivers
Layer 3: Work        — Squad Service, Calcifer, Flywheel, Feedback
Layer 2: Economy     — Treasury, Bank, Bounty Board, $MIND token
Layer 1: Kernel      — SOS (bus, events, registry, governance)
```
