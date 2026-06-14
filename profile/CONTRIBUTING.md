# Contributing to Ergon Automation Labs

Thanks for considering contributing to Bot Army. This document explains how to build bots, propose features, and integrate with the Ergon ecosystem.

---

## What You Can Contribute

### 1. New Bots (Agents)

Solve a problem in your domain by building a bot. Examples:

- **Enterprise**: Service topology mapper, deployment coordinator, team capacity tracker
- **SRE**: Incident responder, log aggregator, blast radius analyzer, runbook bot
- **Personal**: Fitness tracker, learning scheduler, notification router, calendar bot
- **Integration**: GitHub PR analyzer, Slack command responder, external service poller

**Start here:** [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal) — 10-minute template. Build, test, then share.

### 2. Schemas (Message Contracts)

Define NATS message contracts for your bot or domain.

- Define request/response shapes with JSON Schema
- Document domain-specific events (e.g., incident.triggered, deployment.started)
- Enable other bots to integrate with your domain

**Start here:** [ergon-schema-template](https://github.com/ergon-automation-labs/ergon-schema-template) — Simple schema repo scaffolding.

### 3. Integrations

Wire Ergon into your existing infrastructure:

- PagerDuty / Alertmanager (for SRE use cases)
- Kubernetes / Service mesh (for enterprise use cases)
- GitHub / GitLab (for CI/CD)
- Datadog / ELK / Prometheus (for observability)
- Slack / Discord (for messaging)
- Calendar services, task managers, etc.

Integrations live in bots. No separate "integration layer."

### 4. Documentation & Examples

- Runbooks for common scenarios
- Deployment guides for your stack
- Examples of how you integrated with your infrastructure
- "How to get started" guides for your domain

### 5. Infrastructure & Deployment

- Kubernetes manifests for new environments
- Terraform / Helm configurations
- Docker improvements
- Performance optimizations

---

## How to Contribute a Bot

### 1. Identify Your Problem

What decision are you making repeatedly that's stealing your attention?

- Enterprise dev: "Is this safe to deploy?"
- SRE: "What do I do about this alert?"
- Knowledge worker: "What should I work on?"

### 2. Create a Minimal Bot

```bash
git clone https://github.com/ergon-automation-labs/ergon-bot-minimal my-bot
cd my-bot && make setup
```

Your bot now has:
- NATS client and handlers
- Basic database schema (optional)
- Tests
- Docker support
- Makefile targets

### 3. Define Your NATS Subjects

What does your bot respond to? Examples:

```
incident_responder.alert.trigger       → Gather context on incident
incident_responder.status.query        → Check incident status

topology_mapper.service.register       → Register a service
topology_mapper.graph.query            → Query dependencies

fitness_bot.workout.log                → Log a workout
fitness_bot.energy.query               → Get current energy level
```

Document these in your `README.md` and in a schema file.

### 4. Implement Your Logic

- Write handlers for each subject
- Keep logic focused (one bot, one domain)
- Write tests (unit + integration where it makes sense)
- Test with real NATS and real data

### 5. Define Your Schema

Create a `schemas/` directory with JSON Schema files:

```
schemas/
├── incident.trigger.request.schema.json
├── incident.trigger.response.schema.json
├── incident.status.query.schema.json
└── incident.status.query.response.schema.json
```

Register your bot with the NATS registry so decision layer can discover it.

### 6. Document & Share

Write a `README.md` that explains:

- **What problem does this solve?** (2 sentences)
- **What does it do?** (bullet points)
- **Who is it for?** (SRE, enterprise, knowledge workers, etc.)
- **How do I use it?** (example requests)
- **How do I integrate it?** (env vars, dependencies, setup)
- **NATS subjects** (all the endpoints)
- **Database schema** (if applicable)

### 7. Open a PR or Share Your Repo

If you want it in the official organization:
1. Open an issue describing what you built
2. Share your repo link
3. We'll review for: code quality, NATS patterns, schema validity, documentation
4. Once merged, it becomes available to all Ergon users

If you want to keep it private/forked:
- Share it on CONTRIBUTING.md as a community bot
- Other people with your problem can use it

---

## For Enterprise: Building a Service Topology Mapper

**Goal:** Map your service dependencies so the decision layer can analyze blast radius.

**Starting point:**

1. Read [POSITIONING.md → Enterprise](./POSITIONING.md#enterprise-platform-teams--architecture-leads) to understand the problem
2. Fork [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal)
3. Create a handler that reads from your infrastructure:

```elixir
# Your custom bot: topology_mapper
# Reads from: Kubernetes API, Consul, your CMDB, etc.
# Writes to: PostgreSQL graph database (via custom schema)

defmodule TopologyMapper.Handlers.ServiceRegister do
  def handle(%{"service_name" => name, "dependencies" => deps, "owner" => owner}) do
    # 1. Validate the service exists in your infrastructure
    # 2. Write to graph database: service nodes, dependency edges, ownership
    # 3. Return confirmation
  end
end
```

4. Define NATS subjects:
   - `topology_mapper.service.register` — Register a service
   - `topology_mapper.service.query` — Get service details
   - `topology_mapper.blast_radius.query` — What does this service affect?
   - `topology_mapper.owner.query` — Who owns this service?

5. Deploy this bot alongside your CI/CD. On each deploy:
   - Your topology mapper updates the graph
   - Decision layer can now answer: "Safe to merge? Who needs to know?"

6. Share your bot with other platform teams.

---

## For SREs: Building an Incident Responder

**Goal:** When paged, automatically gather context so you can decide fast.

**Starting point:**

1. Read [POSITIONING.md → SREs](./POSITIONING.md#sres--on-call-engineers) to understand the problem
2. Fork [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal)
3. Create a handler that responds to incidents:

```elixir
# Your custom bot: incident_responder
# Reads from: PagerDuty, Alertmanager, Datadog logs, service graph
# Writes to: Contextual incident summaries

defmodule IncidentResponder.Handlers.Alert do
  def handle(%{"alert_id" => id, "service" => service, "severity" => sev}) do
    # 1. Fetch recent deploys to this service
    # 2. Fetch logs from last 30 minutes
    # 3. Query service graph: what does this affect downstream?
    # 4. Who owns this service? Who's on-call?
    # 5. Return summarized context
    
    context = %{
      service: service,
      recent_deploys: [...],
      top_errors: [...],
      affected_downstream: [...],
      owner: "...",
      runbook: "..."
    }
    {:ok, context}
  end
end
```

4. Define NATS subjects:
   - `incident_responder.alert.trigger` — Get incident context
   - `incident_responder.blast_radius.query` — What's affected?
   - `incident_responder.team.query` — Who's on-call?

5. Integrate with your alerting:
   - PagerDuty webhook → NATS publish
   - Alertmanager webhook → NATS publish
   - SRE gets context automatically

6. Share your bot with other on-call teams.

---

## Code Standards

All contributions should follow these patterns:

### Testing

- **Unit tests**: Mock external services, test business logic
- **Integration tests**: Tag with `@tag :integration`, run against real services
- **Run tests before push**: `make test`

Example:
```bash
cd bot_army_my_bot && mix test              # Unit only
cd bot_army_my_bot && mix test --include integration  # With services
```

### Code Quality

- **Compile clean**: `mix compile` with no warnings
- **Credo pass**: `mix credo --strict` (or documented suppressions)
- **Dialyzer pass**: `mix dialyzer` (optional but encouraged)

Run all three:
```bash
make check  # or add to your Makefile
```

### Documentation

- **README.md**: Problem, solution, how to use, NATS subjects
- **Inline comments**: Only for WHY (not WHAT). Code should be clear.
- **Schema files**: JSON Schemas for every NATS subject
- **Runbook** (for ops bots): Steps to deploy, debug, recover

### Naming

- **Bot names**: `<domain>_bot` (e.g., `incident_responder_bot`, `topology_mapper_bot`)
- **NATS subjects**: `<domain>.<verb>.<noun>` (e.g., `topology_mapper.service.register`)
- **Database**: `ergon_<domain>` (e.g., `ergon_topology_mapper`)

---

## Deployment & Maintenance

Once your bot is shared:

1. **Versioning**: Semantic versioning in `mix.exs`. Version bump = potential deployment.
2. **Releases**: Pre-push hooks automatically create GitHub releases on version bumps.
3. **Stability**: Mark your bot's maturity in [BOT_STATUS.md](./BOT_STATUS.md) (production/stable/experimental).
4. **Maintenance**: You own bug fixes and improvements. Community can submit PRs.

---

## Questions?

- **How do I build a bot?** → Start with [ergon-bot-minimal](https://github.com/ergon-automation-labs/ergon-bot-minimal)
- **What's my bot for enterprise?** → See [POSITIONING.md](./POSITIONING.md#enterprise-platform-teams--architecture-leads) + examples above
- **What's my bot for SRE?** → See [POSITIONING.md](./POSITIONING.md#sres--on-call-engineers) + examples above
- **Can I contribute to core bots?** → Yes, open an issue first to discuss the change
- **Can I fork and modify bots?** → Absolutely. Everything is open source (Apache 2.0).

---

## Code of Conduct

Be respectful. Help others learn. Contribute in good faith. No spam, no harassment.

For violations, reach out to the maintainers.

---

## License

All contributions are Apache 2.0. By contributing, you agree to this license.
