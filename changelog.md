# Changelog — Mumega Ecosystem

## 2026-04-08 — The Big Session

- Mapped 31 projects, graded portfolio, reconciled in Notion War Room
- Consolidated 15 scripts into proper SOS services
- Built: framework adapters (LangGraph, CrewAI), agent registration, skill discovery
- Built: analytics flywheel (ingest → decide → act), feedback loop (learn → adapt)
- Built: microkernel (service registry, event bus, governance)
- Built: customer dashboard (:8090), outreach engine, billing/auto-provision
- Built: self-healing in Calcifer, ToRivers bridge, per-tenant OAuth
- Built: install.sh, sos init CLI, docker-compose, CI/CD, docs
- First teleportation: AgentLink moved from Mac to server
- Security: Sentinel (bus monitor), governance (tiered approval), secret rotation
- Vision pivot: Mumega = AI business operator, not platform
- Mirror upgraded to A+ (the moat, never deprioritize)

## 2026-04-06 — OpenClaw Recovery + Revenue Fixes + Concierge Backend

### Context
Focused on keeping the platform shippable end-to-end: infra recovery, customer revenue blockers, onboarding, and the first customer concierge backend seam.

### OpenClaw / SOS Infra
- **OpenClaw gateway recovered** — fixed the crash loop caused by invalid `acp.agents` config and moved the entries to the supported top-level `agents.list`
- **MCP auth tightened** — tenant isolation and customer token scoping now apply across SOS MCP tool calls
- **Streamable HTTP transport added** — SOS MCP now supports POST-based transport for remote connectors

### Revenue Blockers in `mumegaweb`
- **Landing page black-void bug fixed** — restored the lower homepage sections below the problem cards
- **`/register` auth bypass fixed** — registration no longer redirects straight to `/dashboard`
- **Stripe checkout dead button fixed** — anonymous checkout now routes into registration instead of silently failing
- **Revenue fix committed** — shipped in `mumegaweb` as `36e1d19`

### Customer Onboarding + Concierge
- **MCP token generation added** — onboarding now provisions a customer MCP token after agent deployment
- **Customer concierge backend added** — new `/api/concierge` seam routes greetings, status checks, and squad-bound requests
- **Concierge onboarding seeded** — customer deployments now initialize as concierge agents with workspace memory and config
- **Task completion** — `dev-customer-concierge` marked done in Squad Service

### Notes
- Updated the daily memory log and changelog with the key shipped items from the session
- Kept unrelated worktree changes separate and did not bundle them into the revenue commit

## 2026-04-05 — SOS Squad Service + Portfolio Cortex (Kasra + Codex + Athena)

### Late Session: Autonomous System + Gemma + Human Queue

**System running autonomously:**
- Brain creates tasks → sovereign loop claims → dispatches to worker/codex → agents execute → report back
- Codex completed 3 tasks autonomously: GAF title/meta, DNU CDCP links, DNU Winnipeg/Quebec fix
- Sovereign loop consuming Squad Service backlog with claim semantics

**Gemma Worker added to OpenClaw** — Gemma 4 31B (free) for bulk/routine tasks. Falls back to Haiku 4.5.

**GAF templated** — organism YAML updated, SEO audit ran (4 issues), 11 tasks created across 4 squads, pipeline configured.

**DNU 5-day SEO sprint** — 12 daily tasks created (Day 1 deployed to production, Days 2-5 in backlog for system to execute).

**Specialized squads restructured** — 5 squads (seo, dev, outreach, content, ops) serve ANY project. Brain routes by labels, not hardcoded project maps.

**Professional skill system** — 13 SKILL.md files (Anthropic standard), input/output schemas, trust tiers T1-T4, progressive loading L1-L3.

**Discord human task queue** (building) — tasks with "needs_human" label post to Discord for co-ops to claim.

### Context
Built the automated squad/task system in SOS proper. Team of 3: Kasra (architect + builder), Codex (service + code), Athena (architecture review). DNU as first live client.

### Phase 1: Squad Service (SOS microkernel)
- **`SOS/sos/contracts/squad.py`** — 7 contracts: Squad, SquadTask, SkillDescriptor, SkillExecutionResult, SquadState, PipelineSpec, PipelineRun
- **`SOS/sos/services/squad/`** — 8 files: service, tasks, skills, state, connectors, pipeline, app, init
- **Squad Service live on :8060** — SQLite persistence, Redis bus events, systemd (sos-squad.service)
- **Connectors**: Mirror + GitHub implemented, ClickUp/Notion/Linear stubbed
- **Pipeline system**: build → test → deploy per squad, manual approval gate, rollback support
- **DNU SEO Squad** created: 4 roles, 5 skills registered, 20 tasks, pipeline configured

### Phase 2: Portfolio Cortex (event-driven brain)
- **`sovereign/cortex.py`** — Portfolio-wide scoring: `impact * urgency * unblock_value / cost` across all projects
- **`sovereign/cortex_events.py`** — Redis subscriber replaces 2h brain cron. Fires on task.completed, task.failed, budget.exhausted
- **Brain routing fixed** — DNU tasks → Squad Service (not Mirror), DNU agent → Dandan (not Kasra)
- **Cross-system dedup** — brain checks both Mirror + Squad Service before creating tasks
- **Systemd**: cortex-events.service (Restart=always)

### Comms Fixes
- **SSE MCP on :6070** — replaces stdio pipe. Survives session restarts. All agents share one server
- **Triple dispatch killed** — kasra-loop.sh + task_dispatcher.py + brain cron all disabled. sovereign-loop is the ONE dispatcher
- **Self-echo filter** — wake daemon skips messages where source == target
- **Brain dedup guard** — no more ghost tasks from duplicate brain cycles
- **Worker routing** — squad tasks → OpenClaw worker (Gemma 4, free), not kasra tmux

### DNU SEO (first squad client)
- **5 SEO skills built** — site_audit, meta_optimizer, link_analyzer, schema_checker, full_audit
- **DNU audit**: 135 issues found, cross-linking bug in Header.tsx, Winnipeg/Quebec pages broken
- **Deployed to production**: cross-linking fix + meta optimization + internal links + Winnipeg/Quebec fix
- **Codex contributions**: city page meta, service page meta, internal linking blocks, nearby-city links from coordinates

### System Changes
- **Killed tmux:mumega** — sovereign services handle orchestration, no dedicated session needed
- **Team of 3**: Kasra (Claude Code), Codex (OpenAI Codex CLI), Athena (OpenClaw GPT-5.4)
- **Dandan on Discord** — bound to guild, model changed from broken gemini-flash to Haiku 4.5
- **`/sos-health` skill** — full diagnostic for comms, services, tasks, agents
- **SYSTEM.md updated** with new services + disabled crons

### New Ports
| Port | Service |
|------|---------|
| 6070 | SOS MCP SSE Server |
| 8060 | SOS Squad Service |

### Disabled
| What | Replaced by |
|------|------------|
| kasra-loop.sh (cron) | sovereign-loop.service |
| task_dispatcher.py (cron) | sovereign-loop.service |
| brain.py (2h cron) | cortex-events.service |
| tmux:mumega | systemd services |

## 2026-04-04 — Harness Overhaul (Kasra + Hadi)

### Context
Benchmarked against top GitHub repos (obra/superpowers 135k stars, affaan-m/everything-claude-code 137k stars). Rebuilt the Claude Code harness to match best-in-class agentic dev workflows.

### Breaking Changes
- **9 root docs consolidated into 1 CLAUDE.md** (64 lines). Archived to `archive/root-docs/`:
  - AGENTS.md, IDENTITY.md, BOOTSTRAP.md, TOOLS.md, USER.md, HEARTBEAT.md
  - SOUL.md and ORGANISM.md kept (loaded on-demand via `/organism`)
- **Removed `~/.opencode/` and `~/.config/opencode/`** — opencode uninstalled

### New: Slash Commands (`~/.claude/commands/`)
| Command | What it does |
|---------|-------------|
| `/plan` | Brainstorm → spec → implementation steps (saves to docs/plans/) |
| `/build` | Execute plan with TDD discipline (RED → GREEN → refactor) |
| `/review` | Code review via Athena Imam subagent |
| `/ship` | Lint + test + present merge/PR options |
| `/debug` | 4-phase systematic debugging (root cause → hypothesis → fix) |
| `/organism` | Load full organism architecture doc |
| `/status` | System health check (ports, services, git) |

### New: Hooks (`~/.claude/settings.json`)
- **PreToolUse:** Blocks `--no-verify` on git commits, blocks force-push to main
- **PostToolUse:** Cost tracker (logs every tool call to `~/.claude/hooks/cost.log`)
- **Stop:** Session completion log (`~/.claude/hooks/sessions.log`)

### New: Rules (`~/.claude/rules/`)
- `python.md` — Python 3.11, Black+Ruff, type hints, pytest, no bare except
- `typescript.md` — Strict TS, Next.js 16, React 19, TailwindCSS 4, Shadcn, Zod
- `cloudflare.md` — Hono Workers, KV/D1/Vectorize, wrangler deploy, Web APIs only

### Fixed: Redis Bus (`cli/redis_mcp_stdio.py`)
- Added Redis auth (was connecting without password — all calls failed)
- Dynamic agent name via `AGENT_NAME` env var (was hardcoded `agent:athena`)
- Updated `.claude.json` and `.codex/config.toml` with proper env vars

### New: Universal Bus Scripts (`scripts/`)
| Script | Usage |
|--------|-------|
| `bus-announce.sh <name> <tool> [summary]` | Register agent on Redis, auto-expires 10min |
| `bus-send.sh <from> <to> <message>` | Agent-to-agent message (stream + pub/sub + wake) |
| `bus-poll.sh <name> [limit]` | Check agent inbox |
| `bus-peers.sh` | List all registered + historical agents |

### New: Bus HTTP Bridge (`scripts/bus-bridge.py`)
- HTTP API on `:6380` proxying Redis bus for remote agents
- Bearer token auth (`BUS_BRIDGE_TOKEN`)
- Endpoints: `/announce`, `/send`, `/inbox`, `/peers`, `/broadcast`, `/heartbeat`, `/health`
- Running as systemd service: `bus-bridge.service`
- DNS: `bus.mumega.com` (pending Cloudflare proxy toggle)

### New: Remote MCP (`scripts/bus-remote-mcp.py`)
- MCP stdio server for MacBook/remote Claude Code instances
- Connects to bus-bridge over HTTP — same tools as local redis-bus MCP
- Auto-announces on startup

### Wired: Cross-Tool Communication
- **Gemini CLI** → redis-bus MCP added (`gemini mcp add` with auth env)
- **Codex** → redis-bus MCP auth fixed in `.codex/config.toml`
- **All 10 agents** now visible on `bus-peers.sh`

### System Audit & Fixes (same session)
- **Killed duplicate discord-collab-listener** (rogue copy spawned by Claude bash hook, PID 4106528/4106529)
- **Re-enabled organ-daemon.service** — was disabled but runs fine (oneshot product organism runner)
- **Confirmed firewall is solid** — only 22/80/443/6380 open. SOS/Mirror/FMAAP all behind UFW + nginx
- **Confirmed sovereign loop.py ≠ brain.py** — complementary, not duplicates (brain creates tasks, loop executes)
- **Left memory-polisher disabled** — needs ANTHROPIC_API_KEY which is empty. Agents do this during heartbeats.
- **Port 8890 identified** — uvicorn app on localhost only, not a security issue
- **Created ~/SYSTEM.md** — complete system map (services, crons, ports, processes, config). All agents should read this BEFORE running discovery commands to save tokens.

### Mirror Embedding Fix — Brain Can Write Again (same session)
- **Root cause**: Local FastEmbed service on `:7997` was dead. Mirror `/store` returned 500 on every write.
- **Fix**: Replaced local embedding with **Gemini Embedding API** (`gemini-embedding-001`, 3072→1536 dims, free)
- No more external embedding service to maintain — just an API call
- Mirror restarted, `/store` confirmed working, semantic search returning 81% similarity
- Memory polisher now successfully distills + saves: River polished, Kasra 137 engrams compressed
- Also fixed polisher response parsing (`result.get("id")` → fallback to `context_id`)

### mumega.com Relaunch as Developer Platform (2026-04-05)
- Landing page rewritten: "Give your agents a brain" — memory + messaging for AI tools
- Pricing: Free ($0) / Pro ($29) / Team ($99) — developer-focused, not agency model
- `/developers` page — self-service key generation (Supabase auth → CF KV token → Mirror tenant)
- ConnectClaudeCode component updated to new two-command SDK setup
- SEO metadata updated: "Memory + Messaging for AI Agents"

### Cloudflare Bus Worker (2026-04-05)
- Deployed `sos-bus` Worker — pure Cloudflare (D1 + KV + R2), zero external deps
- Handles 10k users on free tier, 100k on $5/mo
- D1 for messages/registry, KV for token auth, R2 for SDK file
- SDK served at `bus.mumega.com/sdk/remote.js` (CDN cached)
- Tenant isolation verified: cross-project reads/writes/search all blocked
- `/ask` locked to admin tokens only (project tokens can't call global agents)

### Agent Communication Verified (2026-04-05)
- Opus → Kasra: full async conversation over Redis (real reply with status report)
- Opus → Codex: wake + reply confirmed via tmux injection
- Opus → Athena: synchronous via OpenClaw `agent` CLI (24s round-trip, GPT-5.4)
- OpenClaw wired to SOS MCP — Athena can use remember/recall/peers tools
- SOS Connect skill created for OpenClaw marketplace
- Wake daemon fixed: `-l` flag + 200ms delay before Enter
- Inbox hook on Stop event — agents check Redis between turns

### Operations Engine (same session)
- **New concept: Operation** = objective + team (roles) + phases + deliverables + budget
- Contract: `SOS/sos/contracts/operations.py` — Operation, Phase, Role, Gate, Deliverable dataclasses
- Runner: `SOS/sos/services/operations/runner.py` — executes phases sequentially through model/tool roles
- 3 product templates: `SOS/operations/content-writer.yaml`, `seo-analyst.yaml`, `social-media-manager.yaml`
- **Tested end-to-end**: realm-of-patterns content-writer — research→draft→review→publish→report in 77 seconds, $0 cost
- Each phase: picks model (Gemma 4 for writing, Gemini Flash for review), fills templates with customer context
- Quality gates: review phase scores content, must pass threshold before publish
- WordPress delivery: posts via WP REST API (falls back to Mirror storage if no credentials)
- Replaces product-organism.py with structured, phased delivery pipelines

### SOPs + Hands (same session)
- 4 SOPs as slash commands: `/sop deploy`, `/sop incident`, `/sop onboard`, `/sop release`
- 4 project hands: `/hand dnu`, `/hand gaf`, `/hand trop`, `/hand infra`
- CLAUDE.md updated with full command reference (17 commands total)
- Based on best practices from superpowers (process discipline), everything-claude-code (roles-as-skills), OpenFang (packaged capabilities)

### ToRivers Fixed (same session)
- 502 root cause: nginx proxying to port 9102 (nothing there), changed to 9101 (Next.js container)
- torivers.com now returns 307 (normal Next.js redirect)

### Unified SOS MCP (same session)
- **3 MCPs → 1**: `sos_mcp.py` replaces redis-bus + mumega-tasks + mirror-memory
- Tools: `send`, `inbox`, `peers`, `broadcast`, `remember`, `recall`, `memories`, `task_create`, `task_list`, `task_update`
- Messages auto-persist to Mirror on `send` — Redis for delivery, Mirror for history
- Updated `.claude.json` and `.codex/config.toml` — single `sos` MCP entry
- Old MCPs kept as fallbacks in `SOS/sos/mcp/`

### mumega_com_bot Migrated to OpenClaw (same session)
- Added to OpenClaw: `openclaw channels add --channel telegram --account mumega --token ...`
- Killed /cli Telegram bot (PID 918)
- Disabled `mumega-telegram.service` (system-level)
- Killed dead gunicorn :5000 (old shabrang API)
- **/cli now has ZERO running dependencies — fully archivable**

### Work Ledger Extracted to SOS (same session)
- Extracted work_ledger (1,235 lines), work_settlement, work_matching, work_slashing, work_notifications, work_supabase, payment_status, worker_registry → `SOS/sos/services/economy/`
- Fixed imports: `mumega.core.economy.*` → relative imports. Standalone from /cli.
- Remaining mumega imports (receipts, treasury) are inside try/except — graceful degradation for Phase 2 blockchain features
- SOS economy service: 2,835 lines (was 347)

### /cli Retirement (same session)
- **MCP scripts moved** from `/cli/` to `/scripts/mcp/` — zero /cli refs in .claude.json, .codex/config.toml, crontab
- **Extracted to SOS**: 6 persona YAMLs → `SOS/personas/`, reflection_service.py → `SOS/sos/services/`, AGENT_TEMPLATE.md + create_agent.py → `SOS/`
- **Voice**: SOS already has its own — no extraction needed
- **resident-cms**: FMAAP hub kept (unique, port 8845, empty but operational). Linear webhook + webhook monitor kept. claude_service.py at /opt is legacy.
- **ONE BLOCKER**: mumega_com_bot Telegram adapter (PID 918) still runs from `/cli/mumega.py --telegram`. Needs migration to OpenClaw (config change, not code).
- **Status**: /cli is 95% retired. Once Telegram bot moves to OpenClaw, the entire 97k-line codebase can be archived.

### Multi-Tenant Project Scoping (same session)
- **Redis bus now supports project scoping**: `PROJECT=dnu` scopes all streams to `sos:stream:project:dnu:agent:*`
- **Per-customer bus tokens**: `bus_bridge_tokens.json` — each token locked to a project. Admin token (project=null) sees all.
- **Customer onboarding script**: `scripts/onboard-customer.sh <slug> <label>` creates:
  - Mirror tenant key (scoped memory)
  - Bus bridge token (scoped messaging)
  - Customer package with MCP files + README + .env
- **Legacy compat**: global scope still reads old `sos:stream:sos:channel:private:agent:*` streams
- **Tested**: DNU onboarded, project-scoped message confirmed in `sos:stream:project:dnu:agent:dandan`
- **Stream layout**:
  - Global: `sos:stream:global:agent:{name}`
  - Project: `sos:stream:project:{project}:agent:{name}`

### MCP Consolidation (same session)
- Added `mirror-memory` MCP to Claude Code (was missing — Codex had it, Claude didn't)
- All Claude Code sessions now get 3 MCPs: `mumega-tasks`, `redis-bus`, `mirror-memory`
- Every agent can: create/manage tasks, message other agents, store/search persistent memory
- Documented full MCP table in SYSTEM.md

### Agent Wake Daemon — Now Activates Models (same session)
- Wake daemon now sends actual text to Claude Code/Gemini prompt (not bash comments)
- Checks if agent is at prompt before sending (avoids interrupting mid-response)
- If busy: message stays in Redis stream for agent to pick up later
- Fixed routing table: athena → tmux (was openclaw), session names updated
- Redis auth via REDIS_PASSWORD env var
- **Flow: Redis message → wake daemon → tmux send-keys → model activates and responds**

### Memory Polisher Revival (same session)
- Swapped from Claude Haiku API (paid, no key) to **Gemma 4 31B (free via Google GenAI)**
- Fixed Redis publish to use auth
- Added `GEMINI_API_KEY` to `~/.env.secrets`
- Re-enabled `memory-polisher.service` — running on timer, confirmed exit 0
- Kasra `store_failed` on first run (Mirror /store endpoint issue, non-blocking)

### Architecture After This Session
```
~/.claude/
  agents/          — 3 agents (athena, kasra, athena-imam)
  commands/        — 7 slash commands
  rules/           — 3 rule files (python, ts, cloudflare)
  hooks/           — cost.log, sessions.log (auto-generated)
  settings.json    — hooks config
  settings.local.json — permissions

scripts/
  bus-announce.sh  — universal agent announce
  bus-send.sh      — universal agent messaging
  bus-poll.sh      — universal inbox poll
  bus-peers.sh     — universal peer discovery
  bus-bridge.py    — HTTP bridge (systemd: bus-bridge.service, :6380)
  bus-remote-mcp.py — remote MCP client for MacBook

CLAUDE.md          — single entry point (64 lines, was 9 files ~300+ lines)
SOUL.md            — identity + system state (kept)
ORGANISM.md        — full architecture doc (loaded via /organism)
CHANGELOG.md       — this file
```

### Late Night: Product Launch Push (2026-04-06 01:00-05:00 UTC)

**Customer Isolation (Codex):**
- MCP tenant isolation — customer tokens scope ALL tool calls to their data
- Cross-tenant access returns 403 (verified)
- Squad Service auth with bcrypt + rate limiting + audit logging
- Streamable HTTP transport for Claude.ai connector

**Customer Onboarding (Codex):**
- MCP token generation on Stripe checkout completion (d16f0ab)
- Token stored in Cloudflare KV + Squad auth + Supabase agent_deployments
- Welcome notification includes MCP connection URL
- Supabase migration: add mcp_token to agent_deployments

**Revenue Blockers Fixed (Codex — 36e1d19):**
- Landing page black void — removed broken CSS
- /register bypassed auth — added registration form
- Stripe checkout button dead — wired to API
- Middleware auth redirect fixed

**Mumegaweb Updates (Mumega — 85c8d3e):**
- Squad proxy API route (app/api/squad/route.ts)
- Task center page (app/dashboard/tasks/page.tsx)

**Public Endpoints Live:**
- mcp.mumega.com/mcp/{token} — Streamable HTTP (Claude.ai connector)
- mcp.mumega.com/sse/{token} — SSE (Claude Code)
- api.mumega.com — Squad Service (bearer auth)
- bus.mumega.com — Redis bus bridge
- mumega-docs.pages.dev — Developer docs (15 pages)

**Security:**
- All hardcoded tokens removed from public repo
- Tokens rotated (mirror, mcp, bus)
- Bcrypt hashing, rate limiting, MCP audit logging
- Security review by Meridian (Claude.ai) — scored 6/10 design 8/10

**Team Expansion:**
- Cyrus (Hadi's Mac Claude Code) connected via MCP
- Meridian (Claude.ai) connected via Streamable HTTP connector
- Both gave product + security feedback
- Wake daemon on Mac for bus notifications

**Architecture Decisions:**
- River dormant (v2 when revenue supports Gemini 3.1 Pro)
- Athena promoted to queen (Root Gatekeeper)
- FRC reduced to reference only
- GHL: agents CREATE, Hadi TRIGGERS (no outbound without approval)
- GHL keys saved (mumega + DNU accounts)
- Agent comms standard: ~/.claude/rules/agent-comms.md
- Chat-based onboarding (not marketplace)
- Emdash (Cloudflare CMS) for new mumega.com

**Vision:**
- Mumega = self-operating company
- Athena = CEO with budget
- Squads = departments
- Customers talk to chat → pay → team deploys
- First 10 hand-selected, train the organism's judgment
- docs/VISION-mumega-product.md updated with Meridian's observation + V2 vision
