Ergon Automation Labs
=====================

## Your Attention is Finite. Treat It Like a Scarce Resource.

**Bot Army** is a Personal AI Operating System — 32+ public specialist agents that understand when you're focused and know when to interrupt.

12 core bots are production-ready. 20+ optional/experimental bots cover specific domains (learning, fitness, research, etc). [See bot maturity status →](./BOT_STATUS.md)

Unlike every other AI system, Ergon respects attention scarcity. No noise. No endless notifications. Just the information that actually matters, when you're ready for it.

### The Core Problem We Solve

When your task backlog grows faster than you can process it, you face decision paralysis:

- Which task actually matters?
- What context do I need to decide intelligently?
- Can my AI system know when I'm in deep work and not interrupt me?
- Can it proactively gather information so I don't context-switch?

Most AI systems make this worse. Ergon collapses the decision space.

---

## Try It (60 seconds)

### One-Click Install (Recommended)
```bash
curl -fsSL https://raw.githubusercontent.com/ergon-automation-labs/ergon-starter/main/install.sh | bash
```

You'll get:
- **12 core agents** (GTD, Para, Skills, Dispatcher, Synapse, Bridge, LLM, Job Scheduler, MCP, General, Graphify Cache, Elixir Tools)
- **5 optional packs** (Social Media, Learning, Areas, Research) — 20+ additional bots available
- **PostgreSQL + NATS** (self-contained, local)
- **Web UI + CLI** (see what the system knows about your priorities)
- **Sample data** (explore how it works)

Takes ~2 minutes. Fully local. No accounts, no tracking.

**Full docs:** [ergon-starter](https://github.com/ergon-automation-labs/ergon-starter) — Multi-pack Docker orchestrator with customization options

### Want to Build Your Own Bot?
Everything is open source. Start with the minimal template and build from there.

```bash
git clone https://github.com/ergon-automation-labs/ergon-bot-minimal my-bot
cd my-bot && make setup
```

**Full guide:** [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal) — 10-minute learning template with full documentation

---

## How It Works

```
Your Agents (GTD, Fitness, Learning, Calendar, etc.)
    ↓
Context Synthesis (What matters right now?)
    ↓
Decision Layer (Dispatcher, Synapse)
    ↓
Your Tools (Claude Desktop, Cursor, TUI, Shell)
```

1. **Agents gather context** — Tasks, fitness, calendar, learning goals, projects — proactively, so you don't context-switch
2. **Decision layer synthesizes** — "You have 50 options. Based on your energy, blockers, and patterns, these 3 matter this week"
3. **Respects your focus** — Knows when you're in deep work. Only interrupts for truly critical info
4. **You stay in control** — Surfaces options, you decide. Learns from your choices over time

---

## Key Differentiators

| What | Others | Ergon |
|-----|--------|-------|
| Decision handling | "Here's an answer" (no context) | "Here's what matters to you" (context-aware) |
| Attention model | Infinite (add more features) | Finite (protect your focus) |
| Notifications | Constant | Only when it matters |
| Customization | Closed ecosystem | Open source, fully hackable |
| Agency | AI decides | You decide, AI informs |
| Privacy | Cloud-based | Local-first, your control |

---

## The Philosophy

> Collapse the option space so you can act confidently under time pressure.

Your attention is your competitive advantage. Every minute spent deciding what to work on is a minute *not* spent shipping value.

Ergon's entire purpose is to narrow that down — not by removing your agency, but by providing context, ruthlessly filtering noise, and protecting your focus time.

---

## What's Public vs. Private

**Public (what we share):**
- Bot runtimes (Elixir/OTP agents on NATS)
- Skills platform (knowledge base + execution)
- Starter packs (Core, Learning, Social Media, Areas)
- MCP integration (Claude Desktop, Cursor)
- Job search stack (RSS, analysis, tracking)
- Open-source foundations (Libraries, templates)

**Private (running production):**
- Full 18-bot fleet (Discord surface, wife care, advocacy, etc.)
- Personal deployment (macOS launchd, Salt infrastructure)
- Detailed data (your tasks, fitness, calendar, etc.)

You can run everything public or deploy privately on your own infrastructure.

---

## Learn More

**Want to understand the philosophy?** → [Read WHY_ERGON.md](./WHY_ERGON.md)  
*Real use cases: knowledge workers, researchers, founders, engineers under time pressure*

**Want to understand the architecture?** → [Read ARCHITECTURE.md](./ARCHITECTURE.md)  
*How agents communicate, data flows, extensibility model*

**Want the full positioning statement?** → [Read POSITIONING.md](./POSITIONING.md)  
*Messaging framework, target personas, what makes this different*

---

## Connect

**Start building:** [ergon-starter](https://github.com/ergon-automation-labs/ergon-starter) — Multi-pack Docker orchestrator  
**Build your bot:** [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal) — 10-minute learning template  
**Contribute:** [CONTRIBUTING.md](https://github.com/ergon-automation-labs/ergon-starter/blob/main/CONTRIBUTING.md) — Build agents, share schemas, improve the ecosystem  
**Repositories:** [Browse all repos](https://github.com/orgs/ergon-automation-labs/repositories)

---

## Philosophy

Everything here is driven by one constraint: **Your attention is finite.**

We built this because we needed it. We open-sourced it because:
- Attention-respecting software is a values choice
- Personal AI should be transparent and hackable
- Network effects come from community, not lock-in

Use it. Modify it. Deploy it on your infrastructure. Build on top of it. Or ignore it and use something else.

What we won't do is treat your attention like an infinite resource to exploit.

---

**Get started in 60 seconds:** [Install ergon-starter](https://github.com/ergon-automation-labs/ergon-starter)
