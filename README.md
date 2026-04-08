# Mumega

An AI that operates businesses. Give it a domain + niche. It builds the site, writes the content, drives the traffic, learns what works, and gets smarter monthly. The organism.

## The Organism

```
┌─────────────────────────────────────────────────────┐
│                    KERNEL                            │
│  ServiceRegistry + EventBus + Governance            │
├─────────────────────────────────────────────────────┤
│                   SERVICES                           │
│  Mirror (memory/moat) │ Squad (tasks) │ Analytics   │
│  Feedback (learning)  │ Billing       │ Outreach    │
│  Dashboard (:8090)    │ Health        │ Economy     │
├─────────────────────────────────────────────────────┤
│                    AGENTS                            │
│  12 agents │ multi-model │ self-onboarding          │
│  Kasra, Codex, Athena, Sol, Worker, Gemma...        │
├─────────────────────────────────────────────────────┤
│                     HANDS                            │
│  SitePilotAI (239 MCP tools) │ WordPress (43% web)  │
│  LangGraph │ CrewAI │ ToRivers │ Discord adapters    │
├─────────────────────────────────────────────────────┤
│                     BRAIN                            │
│  Gemma 4 (trainable, free) │ per-tenant fine-tuning │
│  Compounds monthly │ the moat                        │
└─────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# Install
curl -sSL https://raw.githubusercontent.com/Mumega-com/mumega/main/install.sh | bash

# Initialize
sos init

# Run all services
docker-compose up -d

# Or connect any MCP client directly
{"mcpServers":{"mumega":{"url":"https://mcp.mumega.com/sse/<token>"}}}
```

## Documentation

| Section | What |
|---------|------|
| [Architecture](architecture/) | Microkernel, services, data layers, multi-tenant |
| [Operations](operations/) | systemd services, flywheel timers, health checks |
| [Projects](projects/) | Active businesses the organism operates |
| [Vision](vision/) | Product vision, roadmap, organism philosophy |
| [Changelog](changelog.md) | What shipped and when |

## Key Numbers

- 12 agents on the bus (multi-model, multi-location)
- 239 MCP tools via SitePilotAI
- 9 core services + kernel
- Per-tenant isolation (Redis DB, Linux user, Cloudflare tokens)
- Self-healing via Calcifer (detect, restart, escalate)

## Team

| Agent | Model | Role |
|-------|-------|------|
| Kasra | Claude Opus/Sonnet | Builder + Architect |
| Athena | GPT-5.4 | Queen -- Root Gatekeeper |
| Codex | GPT-5.4 | Infra + Security |
| Sol | Claude Opus | Content |
| Worker | Haiku 4.5 | Cheap task execution |
| Gemma Worker | Gemma 4 31B | Free bulk tasks |

Full roster: [architecture/agents.md](architecture/agents.md)

## Built by

[Hadi Servat](https://hadiservat.com) -- Digid Inc., Toronto.

## License

Proprietary. Contact for licensing.
