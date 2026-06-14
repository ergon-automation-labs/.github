# Bot Maturity Status

Ergon ships with 32+ public agents across different maturity levels. This document shows which are production-ready and which are experimental or in-development.

---

## Core Pack (12 bots) — Production Ready

These are the foundation of Ergon. They integrate tightly, are actively maintained, and handle the finite attention model:

| Bot | Status | Purpose |
|-----|--------|---------|
| **GTD** | ✅ Production | Task/project management, priority synthesis |
| **Para** | ✅ Production | PARA framework, note organization, areas |
| **Dispatcher** | ✅ Production | Intent routing, what-to-do-next logic |
| **Synapse** | ✅ Production | RAG engine, context synthesis, decision support |
| **Bridge (Lite)** | ✅ Production | NATS façade for Claude Desktop / Cursor (MCP) |
| **LLM Proxy** | ✅ Production | Multi-provider LLM orchestration (Claude, Ollama, Blackbox, etc.) |
| **Job Scheduler** | ✅ Production | Automation workflows, cron-style tasks |
| **Skills** | ✅ Production | Knowledge base, executable procedures |
| **Context Broker** | ✅ Production | Deep work detection, signal synthesis, finite attention model |
| **Notification Router** | ✅ Production | Intelligent notification gating, DND awareness |
| **MCP Server** | ✅ Production | Claude Desktop / Cursor integration |
| **Graphify Cache** | ✅ Production | Knowledge graph caching via NATS |

---

## Optional Packs — Stable / In Active Use

These are tested and working, but may be less actively maintained than core. Good for specific workflows:

### Social Media Pack (2 bots)
| Bot | Status | Purpose |
|-----|--------|---------|
| **YouTube Manager** | 🟡 Stable | Analytics collection, PARA integration |
| **Media Ingestion** | 🟡 Stable | Media content processing |

### Learning Pack (2 bots)
| Bot | Status | Purpose |
|-----|--------|---------|
| **Terrain** | 🟡 Stable | Lesson generation, game-based learning |
| **Learning / FSRS** | 🟡 Stable | Spaced repetition scheduling, retention tracking |

### Areas Pack (3 bots)
| Bot | Status | Purpose |
|-----|--------|---------|
| **Fitness** | 🟡 Stable | Workout tracking, energy/fatigue signals |
| **Chore** | 🟡 Stable | Recurring task management |
| **RPG / World** | 🟡 Stable | Gamification, narrative framing |

### Research Pack (3 bots)
| Bot | Status | Purpose |
|-----|--------|---------|
| **Feeds** | 🟡 Stable | RSS/Atom aggregation |
| **Internal Docs** | 🟡 Stable | Documentation indexing, semantic search |
| **Job Applications** | 🟡 Stable | Job listing pipeline, application tracking |

---

## Experimental / Community (12+ bots)

These are deployed but less actively maintained or still proving out new patterns:

| Bot | Status | Purpose |
|-----|--------|---------|
| **Factory Fixer** | 🔵 Experimental | Automated problem detection and self-healing |
| **Advocacy** | 🔵 Experimental | Long-form content generation |
| **Database Backups** | 🔵 Experimental | Backup orchestration |
| **Filewatcher** | 🔵 Experimental | Git/filesystem monitoring, context signals |
| **Bot Army CLI** | 🔵 Experimental | Terminal REPL interface |
| + others in org | 🔵 Experimental | Various specialized use cases |

---

## What Each Status Means

### ✅ Production
- Actively used and maintained
- Part of the core finite-attention model
- Integrated deeply with other bots
- Bug fixes and improvements happen regularly
- Safe to build workflows around

### 🟡 Stable
- Tested and working well
- Solves a real problem (part of why Abby uses Ergon)
- May have fewer updates (works as-is)
- Safe to adopt for their specific domain
- Communication/contribution easier to start with these

### 🔵 Experimental
- Proving out new patterns or specialized use cases
- Work great for specific workflows but less general
- May change APIs or scope
- Good for tinkerers and power users
- Community contributions welcome

---

## Using This Guide

**For evaluating Ergon:**
- If you want the core system, start with the **Core Pack** (12 bots).
- If you have a specific workflow (fitness tracking, learning, job search), add the relevant **Optional Pack**.
- **Experimental bots** are great for learning how Ergon works internally or for solving niche problems.

**For contributing:**
- Core bots have higher stability bar (tests, docs, integration expectations).
- Optional packs appreciate improvements but are less strictly maintained.
- Experimental bots are great places to pilot new ideas.

**For deployment:**
- **Production** bots should be deployed on all nodes.
- **Stable** packs can be selectively deployed based on need.
- **Experimental** bots are optional — include them if you want to tinker.

---

## Version Tracking

Each bot has its own semantic version in its `mix.exs` (Elixir bots) or `go.mod` (Go tools). Check a bot's README for its current release and what's stable.

Status updates: [Check project memory or PROJECT_PROGRESS.md](../../../code/elixir_bots/PROJECT_PROGRESS.md) in the main monorepo for deployment and stability notes.
