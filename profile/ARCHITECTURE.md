# Ergon Architecture — Personal AI Operating System

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR TOOLS & SURFACES                         │
│  Claude Desktop (MCP) │ Cursor (MCP) │ Shell │ TUI │ Web UI     │
└────────────┬──────────────────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DECISION LAYER                                │
│  Context Broker (what's relevant)                               │
│  Dispatcher (what matters now)                                   │
│  Synapse (RAG + decision synthesis)                             │
└────────────┬──────────────────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                  DOMAIN AGENTS (18+)                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ 32+ Public Agents                                      │    │
│  │ Core Pack (12): GTD, Para, Skills, Dispatcher,        │    │
│  │ Synapse, Bridge, LLM, Job Scheduler, MCP, General,    │    │
│  │ Graphify Cache, Elixir Tools                          │    │
│  │                                                        │    │
│  │ Optional Packs (20+): YouTube Manager, Media,         │    │
│  │ Terrain, Internal Docs, Fitness, Chore, RPG,         │    │
│  │ Feeds, RSS, Job Applications, and more…              │    │
│  └────────────────────────────────────────────────────────┘    │
└────────────┬──────────────────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                   NATS MESSAGING BACKBONE                        │
│         Event-driven, multi-cluster, subject-based              │
│    Connects agents, services, and external integrations         │
└────────────┬──────────────────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                   DATA & STATE LAYER                             │
│  PostgreSQL (primary data)                                       │
│  PostgreSQL + pgvector (semantic search)                        │
│  PostgreSQL + Apache AGE (knowledge graph)                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Principles

### 1. Decision Support, Not Automation
Agents surface options and context. *You* make the final decision. This preserves your agency and lets you learn from outcomes.

### 2. Finite Attention Awareness
The system models when you're focused, when you're available, and what's actually high-signal for you personally.

- **Deep work detection** — Git activity, calendar blocks, explicit DND signals
- **Notification gating** — Only interrupt for truly critical information
- **Pull vs. Push** — Users control when they engage, agents don't bombard them

### 3. Modular, Extensible Architecture
- Each agent is independent (separate git repo, separate deployment)
- NATS messaging decouples agents from each other
- You can remove agents you don't need, or add custom ones
- Full source code — no black boxes

### 4. Context Synthesis
No agent works in isolation. Synapse (RAG engine) synthesizes information across domains:
- "Which task should I prioritize?" — needs GTD context + fitness (am I fatigued?) + calendar (am I blocked?)
- "Should I learn this?" — needs learning history + fitness + task backlog
- Smart decisions require *context*

---

## Component Roles

### Data Collection Layer
These agents proactively gather information so you don't context-switch to find it:

| Agent | Collects | Surfaces |
|-------|----------|----------|
| **GTD** | Tasks, projects, goals | Priorities, blockers, decompositions |
| **Fitness** | Workouts, sleep, energy | Energy levels, fatigue risk, recovery needs |
| **Learning** | Study records, spaced repetition | Review recommendations, learning gaps |
| **Job Applications** | Job postings, applications, outreach | Pipeline status, follow-ups due |
| **Calendar** | Meetings, focus blocks, availability | Schedule conflicts, deep work windows |
| **Para** | Notes, Areas, Resources | Connected context, relationships |
| **Internal Docs** | Documentation, wikis, guides | Searchable knowledge base (semantic) |
| **Job Scheduler** | Automation workflows, cron jobs | Scheduled tasks, task execution |

### Decision Layer

| Agent | Input | Output |
|-------|-------|--------|
| **Context Broker** | Data from collectors | "What's relevant right now?" |
| **Dispatcher** | Context signals | "What should you focus on?" |
| **Synapse** | All domain context + embeddings | Synthesized answers, ranked options |
| **Skills** | User requests + knowledge base | Executable actions, step-by-step guidance |

### Integration Layer

**Surfaces** (how you interact):
- Claude Desktop / Cursor (MCP protocol)
- Terminal (shell integration, commands)
- Web UI / TUI (rich UI with keyboard control)
- Discord (notifications, status checks)

**External Services**:
- LLM providers (Claude, Ollama, etc.)
- GitHub (commits, PRs, issues)
- YouTube Analytics (channel performance)
- Calendar (Google Calendar, Outlook)

---

## Information Flow: Example Decision

**You ask: "What should I work on right now?"**

```
1. NATS: bridge.query → "What should I work on?"
   ↓
2. SYNAPSE receives query, fetches:
   - GTD: Top 5 priorities, blockers
   - Fitness: Energy level (are you fatigued?)
   - Calendar: Next 2 hours (any meetings?)
   - Internal Docs: Any relevant context for top priorities
   - Learning: Any urgent reviews due?
   ↓
3. SYNAPSE synthesizes:
   "You have energy. No meetings for 2 hours. Priority A is
    blocked on Priority B (due today). Priority C aligns with
    your learning goal. Recommended: Work on Priority C for
    60 min, then unblock Priority B."
   ↓
4. You see the recommendation and decide.
   ↓
5. System learns from your choice:
   - Did you follow the recommendation?
   - How long did the task actually take?
   - How satisfied are you with the outcome?
```

This loop *collapses the decision space* from "what should I do?" (infinite) to "should I do A, B, or C?" (finite).

---

## Extensibility

### Add a Custom Bot
```bash
git clone https://github.com/ergon-automation-labs/ergon-bot-minimal my-bot
cd my-bot && ./setup_new_bot.sh
```

You now have:
- NATS subject namespace (`my_bot.*`)
- Handler pattern (request → response)
- Database schema (if needed)
- Test structure
- Deployment config

Register your bot's subjects and capabilities, and it automatically integrates with the decision layer.

### Modify Behavior
Every agent is open source. Want different prioritization logic? Fork GTD, modify, deploy your version. The decision layer will use it.

### Connect External Services
NATS messaging means you can:
- Listen to any subject (get notifications from agents)
- Publish to any subject (trigger actions)
- Build custom workflows that span multiple agents

Example: A custom bot that watches fitness data and adjusts task recommendations.

---

## Deployment Models

### Local (Developer / Tinkerer)
Single machine, all agents in one cluster, PostgreSQL in Docker.
- Install: `ergon-starter`
- Great for: Customizing behavior, understanding the system
- Trade-off: Limited redundancy

### Kubernetes (Production)
Multiple machines, microservices architecture, dedicated data layer.
- Deploy via Salt (infrastructure-as-code)
- Agents spread across nodes for resilience
- Central NATS cluster for messaging
- Separate databases for each agent (or shared, depending on privacy needs)
- Trade-off: More operational overhead

### Hybrid
Personal machine + cloud backup/sync for data.
- Your agents run locally (privacy, control)
- Decisions sync to cloud for continuity
- Access from multiple devices

---

## Why This Architecture?

### Resilience
If one agent crashes, others continue working. The decision layer can degrade gracefully.

### Privacy
You control where data lives. Agents don't send data to external APIs unless you configure them to.

### Extensibility
New agents plug in without changing core code. Modify existing agents without forking the entire system.

### Testability
Each agent is independently testable. Integration tests verify NATS messaging contracts.

### Scalability
NATS messaging scales horizontally. Add more agents, more machines, more processing power without rearchitecting.

---

## Data Flow & Privacy

All data is local by default:
- PostgreSQL runs on your machine (or your controlled infrastructure)
- NATS cluster is internal (not exposed to the internet)
- LLM requests go to your chosen provider (Claude, Ollama, etc.)
- You decide what data leaves your system

No telemetry. No analytics. No vendor tracking.

---

## The Finite Attention Model

At the core of every design decision is this model:

```
Your Attention
  ↓
Used for (highest value):
  - Deep work / creation
  - High-stakes decisions
  - Learning new things
  ↓
Used for (medium value):
  - Routine decisions with clear context
  - Coordination with others
  ↓
Wasted on (lowest value):
  - Decision about what to decide
  - Context-switching to gather info
  - Notifications that could wait
  ↓
Ergon's job:
  - Eliminate the lowest-value uses
  - Provide context for medium-value uses
  - Protect the highest-value uses
```

Every feature is designed around this model. Every notification is gated. Every integration respects this constraint.
