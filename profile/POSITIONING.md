# Ergon Automation Labs — Positioning & Messaging

## North Star
**Your attention is finite. Treat it like the scarce resource it is.**

Every AI assistant pretends you have infinite attention. Notifications pile up. Decisions get deferred. Focus breaks into fragments.

Ergon is different. We built a Personal AI Operating System that *understands attention scarcity* and defends your focus time ruthlessly.

---

## The Problem We Solve

**Decision paralysis under time pressure.**

When your backlog grows faster than you can process it:
- Which task actually matters right now?
- What context do I need to decide intelligently?
- Can my AI assistant know when I'm in deep work and not interrupt me?
- Can it fetch information proactively, so I don't context-switch to gather it?

Most AI systems make this worse. They add more notifications, more options, more things to decide about.

**Ergon collapses the decision space.** It gathers context, prioritizes ruthlessly, and respects your focus time. When it surfaces something, it's because it actually matters — not because a prompt got routed to a model.

---

## Core Positioning

### What It Is
A **distributed personal AI operating system** (32+ public specialist agents on Elixir/OTP + NATS, 12 in core pack) designed for **decision support, not decision automation.**

It runs on your machine, learns your patterns, and integrates with your existing tools (Claude Desktop, Cursor via MCP, your text editor).

### What Makes It Different

**1. Finite Attention Awareness**
- Knows when you're in deep work (via Git activity, calendar, user signals)
- Defers non-critical notifications automatically
- Surfaces high-signal information, suppresses noise
- Push surfaces interrupt only when it matters; pull surfaces wait until you're ready

**2. Extensible Foundation**
- Open source (Apache 2.0), fully hackable
- Build custom bots for your specific workflow
- NATS messaging layer — connect any tool, any service
- Modular design — use what you need, ignore the rest

**3. Context-Aware, Not Context-Naive**
- Synthesizes information from multiple sources (tasks, fitness, calendar, learning, projects)
- Understands relationships between domains
- Makes decisions with fuller information than generic AI assistants
- Learns what "high-signal" means for *you*, not average users

**4. Respects Your Agency**
- Surfaces options, doesn't make decisions for you
- Learns from your choices over time
- No forced integrations or lock-in
- You control which services connect, when

---

## Target Personas

### Primary: Knowledge Workers Under Time Pressure
- Software engineers, researchers, product managers, writers
- 50+ hours/week, deep focus required
- Drowning in information (tasks, emails, notifications, Slack)
- Currently using: Multiple disparate tools (GitHub, Linear, calendar, notes)
- Problem: "I have all this information but no way to synthesize it into good decisions"

### Secondary: Open-Source Community / Platform Builders
- Developers who want to build on top of a solid foundation
- Teams that want to customize behavior for domain-specific workflows
- People skeptical of closed AI ecosystems
- Problem: "Most AI platforms are black boxes. I want something I can understand and extend."

### Enterprise: Platform Teams & Architecture Leads
- Large organizations with 10s-100s of services, complex dependencies
- Juggling not 50 tasks but 50 *decision points* per task (blast radius, team coordination, contracts)
- Currently using: Service meshes, CMDB, scattered runbooks, Slack for coordination
- Problem: "I have full architecture visibility but no way to synthesize it into fast decisions under coordination complexity"
- **Status**: Core architecture supports this. Needs: service mesh graph mapping, team/ownership metadata, deployment coordination bots
- **How to help**: [Build a topology mapper](https://github.com/ergon-automation-labs) that reads your CMDB and syncs to Ergon's graph DB. Show other platform teams how you did it. [Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues) to discuss.

### SREs / On-Call Engineers
- Paged at 2 AM with sparse context, 5 minutes to decide (rollback? patch? wait?)
- Current state: Grepping logs, checking dashboards, trying to remember who wrote this service
- Currently using: PagerDuty, monitoring tools, tribal knowledge, runbooks
- Problem: "I need to gather context and make a decision *right now*, but I'm context-switching across 5+ systems"
- **Status**: Core architecture supports incident response. Needs: log aggregation bot, incident responder (gathers context automatically), blast radius analyzer, runbook/decision support
- **How to help**: [Build an incident responder](https://github.com/ergon-automation-labs) that integrates with your incident management tool (PagerDuty, Opsgenie). Map your services and show the blast radius. [Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues) to discuss patterns.

---

## Messaging Framework

### Emotional Hook
"Your attention is your competitive advantage. Stop letting AI systems treat it like it's infinite."

### Feature → Benefit → Belief

| Feature | Benefit | Belief |
|---------|---------|--------|
| Knows when you're in deep work | Respects your focus time | AI can be considerate |
| Synthesizes data from 5+ sources | Better decisions, less context-switching | Smarter decisions come from context |
| Extensible, open-source | Build what you need, nothing more | You should control your tools |
| NATS messaging backbone | Connect anything, no vendor lock-in | Flexibility beats lock-in |

### Key Phrases
- "Decision support, not automation"
- "Finite attention awareness"
- "Pull when you're ready, push only when it matters"
- "Your rules, not ours"
- "Collapse the decision space"

---

## What We're NOT

- A chatbot that tries to be your best friend
- An automation framework that strips away your agency
- A surveillance system that monitors everything you do
- A closed ecosystem that forces you into specific workflows
- Yet another notification machine

---

## The Ask

**Try it locally in 60 seconds:**
```bash
curl -fsSL https://raw.githubusercontent.com/ergon-automation-labs/ergon-starter/main/install.sh | bash
```

**Want to customize it?** Everything is open source. Fork it, modify it, make it yours.

**Want to contribute?** Build a bot, share a schema, extend the platform.

---

## Why Open Source?

Closed AI systems optimize for vendor growth, not user attention. We built this because we needed it, not because there's a market to extract value from. Open sourcing it means:

- You can see exactly how decisions are made
- You can modify behavior to match your values
- You're not trapped if our priorities diverge from yours
- The community can improve it faster than any single company

---

## Differentiators vs. Alternatives

| Alternative | Strength | Ergon Advantage |
|-------------|----------|-----------------|
| Generic AI Assistants (ChatGPT, Claude) | Easy to use | Understands your workflow, respects attention |
| Task Management Tools (Linear, Notion) | Well-designed UI | Intelligent prioritization, proactive info gathering |
| Automation Frameworks (IFTTT, Zapier) | Simple integrations | Full control, extensible, respectful of human judgment |
| Agentic AI (AutoGPT, LangChain frameworks) | Powerful abstractions | Finite attention awareness, designed for realistic workflows |

---

## Enterprise & SRE: Invitation to Contribute

We've built Ergon for individual knowledge workers. **We haven't yet built for enterprise platform teams or SRE on-call workflows.** But the architecture supports it. If you need it, we'd love for you to build it:

### Enterprise Platform Teams
**The problem you face:** At scale, decision complexity explodes. You're not deciding "what should I work on" — you're deciding "given 10 service dependencies, team ownership, deployment windows, and contract boundaries, what can I change safely?"

**What's needed:**
- Service topology mapper (read from your CMDB/service mesh → populate Ergon's graph DB)
- Ownership/team metadata (who owns what service, who's on-call, team capacity)
- Deployment coordination bot (track deployment windows across regions, prevent unsafe parallel deploys)
- Change impact analyzer (given a code change, what services break, who needs to know)

**How to build it:**
1. Fork [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal)
2. Connect to your service mesh (Kubernetes API, Consul, your CMDB)
3. Build a bot that syncs topology to Ergon's PostgreSQL + Apache AGE graph database
4. Write decision bots that query the graph
5. Share your architecture — other platform teams will use it

**[Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues)** to discuss patterns. We'll help you integrate.

### SREs / On-Call Engineers
**The problem you face:** Paged at 2 AM with sparse context. You have 5 minutes to decide. You're context-switching across logs, dashboards, deployment history, runbooks, and tribal knowledge.

**What's needed:**
- Incident responder bot (get paged → automatically gather relevant context)
- Log aggregation bot (search logs across all services, surfaces top errors)
- Blast radius analyzer (this service affects what downstream, who's the expert)
- Runbook/decision support (not autopilot — surface your options and reasoning)
- Team signal (who's on-call for dependent services, who has context)

**How to build it:**
1. Fork [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal)
2. Integrate with your incident management tool (PagerDuty, Opsgenie, Alertmanager)
3. Build a bot that responds to incidents by gathering context: recent deploys, affected services, related alerts
4. Connect it to your logs (Datadog, ELK, Loki) and dashboards (Prometheus, Grafana)
5. Map your services so blast radius analysis works
6. Share your runbooks — other on-call engineers will benefit

**[Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues)** to discuss patterns. We'll help you build it.

---

## Philosophy

**Collapse the option space so you can act confidently under time pressure.**

When your decision space is infinite, you spend cognitive energy just deciding what to decide about. Ergon's job is to narrow that down to the few options that actually matter, gather the context you need, and get out of the way so you can move.

It's not about doing more. It's about doing what matters.

---

## Bot Maturity & Stability

The 32+ public agents fall into three categories:

- **✅ Production** (12 core bots) — Foundation of the system, actively maintained, fully integrated
- **🟡 Stable** (10+ optional bots) — Tested, working, solving real problems, less actively updated
- **🔵 Experimental** (10+ bots) — Proving patterns, great for learning, community contributions welcome

**Full breakdown:** [BOT_STATUS.md](./BOT_STATUS.md)
