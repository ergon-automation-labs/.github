# Getting Started: Bot Army for Enterprise Platform Teams

You have complex service architectures. Developers need to make decisions safely: "Can I merge this? Who needs to know? What breaks if I deploy now?"

Bot Army can help. Here's how.

---

## The Problem You're Solving

Your platform team makes decisions under complexity:

- **"Is this safe to deploy?"** → Requires knowing: service dependencies, recent changes, on-call status, deployment windows
- **"Who needs to review this?"** → Requires knowing: service ownership, team capacity, approval chains
- **"What's the blast radius?"** → Requires knowing: downstream consumers, API contracts, usage patterns

These decisions are made by grepping dashboards, Slack, and tribal knowledge. **This is stealing your engineers' attention.**

Bot Army collapses this decision space by synthesizing the information automatically.

---

## Architecture: Your Services → Bot Army → Better Decisions

```
Your Infrastructure
├── Kubernetes / Consul / CMDB (service metadata)
├── Git / GitHub (change history)
└── Deployment Pipeline (recent deploys)
    ↓
Service Topology Mapper Bot
(reads your infrastructure, writes to Bot Army graph)
    ↓
PostgreSQL + Apache AGE (knowledge graph)
    ↓
Decision Bots
├── Blast Radius Analyzer ("what's affected?")
├── Deployment Coordinator ("when is it safe?")
├── Team Resolver ("who needs to know?")
    ↓
Your Developers
("what should I do?" → decision + context)
```

---

## Phase 1: Map Your Services (Week 1)

### Goal
Write a bot that reads your service metadata and syncs it to Bot Army's graph database.

### What You'll Build

A custom bot that:
1. Reads from your infrastructure (Kubernetes API, Consul, CMDB, whatever you have)
2. Extracts: service name, owner, dependencies, API contracts
3. Writes to PostgreSQL + Apache AGE (Bot Army's graph database)
4. Publishes NATS events so other bots can query the graph

### Steps

**1. Fork the template**
```bash
git clone https://github.com/ergon-automation-labs/ergon-bot-minimal service_topology_mapper
cd service_topology_mapper
./setup_new_bot.sh
```

**2. Create a handler that reads your infrastructure**
```elixir
# lib/service_topology_mapper/handlers/service_register.ex
defmodule ServiceTopologyMapper.Handlers.ServiceRegister do
  def handle(%{
    "service_name" => name,
    "owner_team" => team,
    "dependencies" => deps,
    "api_version" => version
  }) do
    # 1. Validate service exists in your CMDB
    # 2. Extract dependency graph
    # 3. Write to PostgreSQL graph database:
    #    CREATE (s:Service {name, owner, version})
    #    CREATE (s)-[:DEPENDS_ON]->(d:Service {...})
    # 4. Return confirmation
    
    {:ok, %{"service_registered" => name, "dependencies_count" => length(deps)}}
  end
end
```

**3. Define NATS subjects**
```
service_topology.service.register
service_topology.service.query
service_topology.dependencies.query
service_topology.owner.query
```

**4. Test locally**
```bash
make setup
make test
make run   # local NATS + Postgres
```

**5. Wire into your CI/CD**
On each deploy:
- CI sends: `service_topology.service.register` with service metadata
- Topology mapper updates the graph
- Graph is now current

### Example: Reading from Kubernetes

```elixir
defmodule ServiceTopologyMapper.K8sReader do
  def list_services do
    # 1. Call Kubernetes API
    # GET /api/v1/namespaces/default/services
    # 2. Extract service name, owner (from labels)
    # 3. Extract endpoints (consumers)
    # 4. For each service, return:
    {:ok, [
      %{
        "service_name" => "auth-service",
        "owner_team" => "platform-security",
        "dependencies" => ["db", "cache"],
        "api_version" => "v2",
        "consumers" => ["gateway", "web-api"]
      },
      ...
    ]}
  end
end
```

---

## Phase 2: Build Decision Bots (Week 2-3)

### Goal
Query the service graph to answer: "Is this safe? Who needs to know?"

### What You'll Build

**Bot 1: Blast Radius Analyzer**
```
Query: service_topology.blast_radius.query
Input: {service: "auth-service", change_type: "schema"}
Output: {
  affected_downstream: ["gateway", "web-api"],
  critical: ["payment-service"],
  breaking_change: true,
  teams_to_notify: ["platform-security", "payments"]
}
```

**Bot 2: Deployment Coordinator**
```
Query: service_topology.deployment.safe_now
Input: {service: "web-api"}
Output: {
  safe: true,
  reason: "no active deployments in dependent services",
  in_flight: [],
  wait_minutes: 0
}
```

### Queries Against the Graph

```sql
-- Blast radius: what depends on service X?
MATCH (n:Service {name: 'auth-service'}) -[:DEPENDED_ON_BY]-> (m:Service)
RETURN m.name, m.owner

-- Who owns service X?
MATCH (s:Service {name: 'auth-service'}) -[:OWNED_BY]-> (t:Team)
RETURN t.name

-- Full dependency tree
MATCH path = (s:Service {name: 'web-api'})-[:DEPENDS_ON*]->(d:Service)
RETURN relationships(path), nodes(path)
```

### Implementation Pattern

```elixir
defmodule ServiceTopologyMapper.Handlers.BlastRadius do
  def handle(%{"service" => service_name}) do
    # Query the graph database
    query = """
    MATCH (n:Service {name: $service})
           -[:DEPENDED_ON_BY]-> (m:Service)
    RETURN m.name, m.owner
    """
    
    affected = execute_graph_query(query, %{"service" => service_name})
    
    {:ok, %{
      "affected_downstream" => Enum.map(affected, & &1["name"]),
      "teams_to_notify" => Enum.uniq(Enum.map(affected, & &1["owner"])),
      "critical" => identify_critical(affected)
    }}
  end
  
  defp identify_critical(services) do
    # Services that touch payments, auth, or other sensitive domains
    Enum.filter(services, &is_critical/1)
  end
end
```

---

## Phase 3: Integrate with Your Decision Layer (Week 3-4)

### Goal
Developers get context automatically when making decisions.

### How It Works

**Scenario: Developer opens a PR**

1. **PR webhook** → NATS: `service_topology.pr_context.request`
2. **Bot fetches context**:
   - Git: What changed?
   - Graph: What depends on this?
   - Teams: Who owns downstream services?
   - Deployments: Are any in-flight?
3. **PR comment**: "This PR affects 2 downstream services (owned by Platform team). Last deploy to auth-service was 15 min ago. Safe to merge."

### Integration Points

**GitHub Actions Webhook**
```bash
curl -X POST \
  http://nats-server:4222/publish/service_topology.pr_context.request \
  -d '{
    "repo": "myorg/auth-service",
    "pr_number": 123,
    "files_changed": ["src/handlers.rs"],
    "requested_by": "dev@myorg.com"
  }'
```

**Team Slack Integration**
```bash
# When a critical service is about to deploy
service_topology.deployment.notify_teams
Input: {service: "auth-service", timestamp: "now"}
Output sent to Slack: "auth-service deploying in 5 minutes. 3 teams affected."
```

---

## Phase 4: Share Your Work (Week 4+)

### Document It
```
service_topology_mapper/
├── README.md (how it works, how to deploy)
├── docs/
│   ├── ARCHITECTURE.md (graph schema, queries)
│   ├── INTEGRATION.md (how to wire into your CI/CD)
│   └── EXAMPLES.md (example requests/responses)
├── schemas/
│   ├── service.register.request.schema.json
│   └── blast_radius.query.response.schema.json
└── tests/ (unit + integration)
```

### Share Your Bot
1. Make it public (or share repo link)
2. Other platform teams can fork and customize
3. Submit to [COMMUNITY_BOTS.md](https://github.com/ergon-automation-labs/ergon-starter/blob/main/COMMUNITY_BOTS.md)

---

## Testing Your Topology Mapper

### Unit Tests
```elixir
test "blast_radius returns affected services" do
  # Mock graph queries
  assert BlastRadius.handle(%{"service" => "auth"}) == {:ok, %{
    "affected_downstream" => ["gateway", "web-api"],
    "teams_to_notify" => ["platform"],
    "critical" => ["payment-service"]
  }}
end
```

### Integration Tests
```elixir
@tag :integration
test "reads actual Kubernetes services and writes to graph" do
  # Spin up postgres + AGE
  # Call K8s API
  # Write to graph
  # Query graph
  # Verify
end
```

### Manual Testing
```bash
# Start local Bot Army
docker compose up

# Register a service
nats request --server nats://localhost:4223 \
  service_topology.service.register \
  '{"service_name":"web-api","owner_team":"platform",...}'

# Query blast radius
nats request --server nats://localhost:4223 \
  service_topology.blast_radius.query \
  '{"service":"auth-service"}'
```

---

## Deployment

### Local Development
```bash
make setup          # Install deps
make test           # Run tests
make run            # Start with docker compose
```

### Production Deployment
```bash
# Build release
make release

# Deploy via Salt
salt '*' state.apply bots.service_topology_mapper

# Verify
salt '*' cmd.run 'systemctl status service_topology_mapper'
```

---

## What's Next?

Once your topology is mapped:

1. **Add data flow** — Model not just dependencies but data contracts (what version of the API does each service expect?)
2. **Add team capacity** — Integrate with your ticketing system (are teams overbooked?)
3. **Add SLO tracking** — Wire in your monitoring (is this service meeting its SLO?)
4. **Add approval workflows** — Automatically route PRs to service owners

---

## Need Help?

- **Questions about architecture?** See [ARCHITECTURE.md](./ARCHITECTURE.md)
- **Questions about building bots?** See [CONTRIBUTING.md](./CONTRIBUTING.md)
- **Questions about your specific infra?** [Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues)

Your problem is not unique. Other platform teams are solving the same thing. Let's build this together.
