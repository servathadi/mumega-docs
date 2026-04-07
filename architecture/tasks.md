# Tasks (Squad Service)

Work orchestration. Squads own tasks, agents claim and execute them.

## Stack

- **API**: FastAPI on `:8060` (systemd: `sos-squad.service`)
- **Database**: SQLite at `~/.sos/data/squads.db`
- **Bus events**: Redis pub/sub on task state changes
- **Auth**: Bearer tokens, tenant-scoped, bcrypt hashing

## Core Concepts

### Squads
Isolated project teams. Each squad has: id, name, project, objective, roles, members, KPIs, budget.

### Tasks
Work items with claim semantics:
- **backlog** → **claimed** (atomic, prevents double-dispatch) → **in_progress** → **done** / **failed**
- Priority: low, medium, high, critical
- Labels for routing: `seo`, `dev`, `content`, `outreach`, `ops`
- Blocked-by / blocks for dependencies

### Skills
Registered capabilities with SKILL.md manifests (Anthropic standard):
- Trust tiers: T1 (read-only) → T4 (destructive)
- Loading levels: L1 (title only) → L3 (full context)
- Input/output JSON schemas
- Fuel grade: diesel (free) → aviation (expensive)

### Pipelines
Per-squad lifecycle: build → test → review → deploy → smoke.
Manual approval gates. Rollback support.

## Task Routing

Sovereign Loop (`sovereign/loop.py`) picks up tasks and dispatches:

1. If task has explicit `agent` field → route directly to that agent
2. If task has labels → match to squad type (seo→worker, dev→codex, etc.)
3. Fallback → keyword matching on title/description

## Delivery

When a task completes:
- Result stored in SQLite
- Bus event emitted: `task.completed`
- Customer project agent gets delivery notification in their stream
- Customer checks `inbox()` to see results

## MCP Tools

- `task_create(title, description, project, priority, labels)` — create work
- `task_list(project, status, limit)` — list tasks
- `task_update(task_id, status, notes)` — update status
- `request(description, priority)` — customer intake (auto-routes to right squad)

## REST API

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/squads` | GET/POST | List/create squads |
| `/squads/{id}` | GET/PUT/DELETE | Squad CRUD |
| `/tasks` | GET/POST | List/create tasks |
| `/tasks/{id}/claim` | POST | Atomic claim |
| `/tasks/{id}/complete` | POST | Complete with result |
| `/skills` | GET/POST | Skill registry |
| `/squads/{id}/pipeline` | GET/PUT | Pipeline config |
| `/squads/{id}/pipeline/run` | POST | Trigger pipeline |
