# Getting Started: Bot Army for SREs / On-Call Engineers

You get paged at 2 AM with sparse error info. You have 5 minutes to decide: is this a rollback, a patch, or can we wait?

Right now you're context-switching across 5+ systems (logs, dashboards, deployment history, Slack, runbooks). **You need context fast. Bot Army can gather it automatically.**

---

## The Problem You're Solving

You're on-call. You get paged:

```
P1 Alert: auth-service error rate > 95%
Service: auth-service
Severity: SEV-1 (customer-facing)
Timestamp: 2026-06-15 02:47:33 UTC
```

You need to answer in seconds:
- **What changed?** (Last 3 deploys? Git commits?)
- **What's the blast radius?** (How many customers affected? Which services depend on this?)
- **What do I do?** (Rollback? Patch? Wait for dawn?)
- **Who do I call?** (Who owns this service? Who's online?)

Right now you're grepping logs, checking dashboards, finding the runbook, Slack-messaging the service owner. **By the time you have context, 10 minutes have passed. Your customers have felt the impact.**

---

## Architecture: Alert → Instant Context → Faster Decisions

```
PagerDuty / Alertmanager / Your Monitoring
    ↓
Incident Responder Bot
(gathers context automatically in parallel)
    ├─→ Fetch Recent Deploys (Git API)
    ├─→ Fetch Logs (Datadog / ELK / Loki)
    ├─→ Fetch Service Topology (service graph)
    ├─→ Fetch On-Call Status (PagerDuty / Opsgenie)
    └─→ Fetch Runbook (internal docs)
    ↓
Synthesized Context
├── What changed: "Deploy to auth 12 min ago (JWT refactor)"
├── Blast radius: "Payment service depends on auth (CRITICAL)"
├── On-call: "Alice owns auth-service, Bob owns payment-service (also on-call)"
└── Runbook: "See: /docs/auth-service/deployment-rollback.md"
    ↓
You ("rollback the JWT refactor")
Decision in 30 seconds instead of 10 minutes.
```

---

## Phase 1: Build Your Incident Responder Bot (Week 1)

### Goal
When paged, gather context automatically from all your systems.

### What You'll Build

A bot that:
1. Gets paged (PagerDuty webhook or Alertmanager alert)
2. Reads from: Git, logs, monitoring, service graph, runbooks
3. Synthesizes into a single response with context
4. Returns: what changed, blast radius, what to do

### Steps

**1. Fork the template**
```bash
git clone https://github.com/ergon-automation-labs/ergon-bot-minimal incident_responder
cd incident_responder
./setup_new_bot.sh
```

**2. Create a handler that responds to alerts**
```elixir
# lib/incident_responder/handlers/alert.ex
defmodule IncidentResponder.Handlers.Alert do
  def handle(%{
    "service" => service_name,
    "alert_id" => alert_id,
    "severity" => severity,
    "error_message" => error_msg
  }) do
    # Gather context in parallel
    context = Task.await_many([
      Task.async(fn -> fetch_recent_deploys(service_name) end),
      Task.async(fn -> fetch_logs(service_name, last: "30m") end),
      Task.async(fn -> fetch_blast_radius(service_name) end),
      Task.async(fn -> fetch_on_call(service_name) end),
      Task.async(fn -> fetch_runbook(service_name) end)
    ])
    
    # Synthesize into actionable context
    {:ok, %{
      "service" => service_name,
      "alert_id" => alert_id,
      "severity" => severity,
      "recent_deploys" => context.deploys,
      "blast_radius" => context.affected_services,
      "on_call" => context.engineers,
      "top_errors" => context.logs,
      "runbook_url" => context.runbook,
      "recommendation" => synthesize_recommendation(context)
    }}
  end
  
  defp synthesize_recommendation(context) do
    cond do
      is_critical_blast_radius?(context) ->
        "CRITICAL: Blast radius includes payment service. Recommend immediate rollback."
      is_recent_deploy?(context, minutes: 5) ->
        "Deploy within last 5 minutes. Check deploy diff: #{context.deploy_diff_url}"
      has_known_issue?(context) ->
        "Known issue: #{context.known_issue}. See runbook: #{context.runbook_url}"
      true ->
        "No obvious cause. Check logs for patterns. Escalate if SLO impacts."
    end
  end
end
```

**3. Define NATS subjects**
```
incident_responder.alert.trigger          → Gather context on incident
incident_responder.incident.query          → Get incident details
incident_responder.blast_radius.query      → What's affected?
incident_responder.team.query              → Who's on-call?
incident_responder.runbook.query           → Get runbook URL
```

**4. Integrate with your alerting system**

**PagerDuty Webhook:**
```bash
# In PagerDuty: Services → Integrations → Webhooks
# Webhook URL: https://your-nats-gateway/publish/incident_responder.alert.trigger

# Webhook payload format:
{
  "alerts": [{
    "alert_key": "...",
    "status": "triggered",
    "service": "auth-service",
    "urgency": "high",
    "html_url": "..."
  }]
}
```

**Alertmanager:**
```yaml
# alertmanager.yml
route:
  receiver: 'bot_army'

receivers:
  - name: 'bot_army'
    webhook_configs:
      - url: 'http://nats-gateway:8080/publish/incident_responder.alert.trigger'
```

**Manual Trigger (for testing):**
```bash
nats request --server nats://localhost:4223 \
  incident_responder.alert.trigger \
  '{
    "service":"auth-service",
    "alert_id":"12345",
    "severity":"SEV-1",
    "error_message":"JWT validation failing"
  }'
```

---

## Phase 2: Wire in Your Data Sources (Week 2)

### Goal
Connect your incident responder to all the systems that hold context.

### What to Connect

**1. Git / GitHub**
```elixir
# Get recent deploys to this service
def fetch_recent_deploys(service_name, limit: 3) do
  {:ok, [
    %{"commit" => "a1b2c3d", "message" => "JWT refactor", "author" => "alice@company.com", "deployed_at" => "2026-06-15 02:35:00"},
    %{"commit" => "e4f5g6h", "message" => "Add caching", "author" => "bob@company.com", "deployed_at" => "2026-06-15 00:15:00"},
    ...
  ]}
end
```

**2. Logs (Datadog / ELK / Loki)**
```elixir
# Get top errors in the last 30 minutes
def fetch_logs(service_name, last: "30m") do
  {:ok, [
    %{"error" => "JWT decode failed", "count" => 15000, "first_seen" => "2026-06-15 02:35:10"},
    %{"error" => "HMAC verification failed", "count" => 14800, "first_seen" => "2026-06-15 02:35:15"},
    ...
  ]}
end
```

**3. Service Graph (from your topology mapper)**
```elixir
# What services does this depend on?
# What services depend on this? (BLAST RADIUS)
def fetch_blast_radius(service_name) do
  {:ok, %{
    "depends_on" => ["db-postgres", "cache-redis"],
    "depended_on_by" => ["payment-service", "web-api"],
    "critical_downstreams" => ["payment-service"],
    "estimated_customers_affected" => 5000
  }}
end
```

**4. On-Call Status (PagerDuty / Opsgenie)**
```elixir
# Who owns this service? Who's online?
def fetch_on_call(service_name) do
  {:ok, %{
    "primary_owner" => "Alice Singh",
    "slack" => "@alice",
    "on_call_now" => true,
    "backup_owner" => "Bob Chen",
    "backup_on_call" => false,
    "team_slack_channel" => "#platform-security"
  }}
end
```

**5. Runbooks (Internal Docs / Git Wiki)**
```elixir
# Where is the runbook for this service?
def fetch_runbook(service_name) do
  {:ok, %{
    "url" => "https://docs.company.com/runbooks/auth-service",
    "deployment_rollback" => "https://docs.company.com/runbooks/auth-service#rollback",
    "jwt_issues" => "https://docs.company.com/runbooks/auth-service#jwt-validation"
  }}
end
```

### Integration Pattern

Each data source is a separate module:

```
lib/incident_responder/
├── data_sources/
│   ├── git_client.ex (GitHub API)
│   ├── log_aggregator.ex (Datadog / ELK)
│   ├── service_graph.ex (from bot_army graph database)
│   ├── on_call_status.ex (PagerDuty API)
│   └── runbook_client.ex (internal wiki / docs)
└── handlers/
    └── alert.ex (orchestrates all data sources)
```

---

## Phase 3: Test End-to-End (Week 2-3)

### Unit Tests
```elixir
test "synthesizes critical alert context" do
  context = %{
    deploys: [%{"message" => "JWT refactor"}],
    logs: [%{"error" => "JWT decode failed", "count" => 15000}],
    affected: ["payment-service"],
    on_call: %{"name" => "Alice"}
  }
  
  rec = IncidentResponder.synthesize_recommendation(context)
  assert String.contains?(rec, "rollback")
end
```

### Integration Tests
```elixir
@tag :integration
test "end-to-end: alert triggers, context gathered, recommendation given" do
  # 1. Trigger alert via NATS
  response = nats_request("incident_responder.alert.trigger", %{
    "service" => "auth-service",
    "alert_id" => "12345"
  })
  
  # 2. Verify response has all context
  assert response["recent_deploys"]
  assert response["blast_radius"]
  assert response["on_call"]
  assert response["runbook_url"]
  assert response["recommendation"]
  
  # 3. Verify recommendation is actionable
  assert String.length(response["recommendation"]) > 0
end
```

### Manual Testing
```bash
# Start local setup
docker compose up

# Send a test alert
nats request nats://localhost:4223 \
  incident_responder.alert.trigger \
  '{
    "service":"auth-service",
    "alert_id":"test-12345",
    "severity":"SEV-1"
  }' \
  --timeout 5s

# Verify response includes:
# - recent_deploys
# - blast_radius
# - on_call status
# - runbook URL
# - recommendation
```

---

## Phase 4: Production Integration (Week 3-4)

### Deploy Your Bot
```bash
make release
salt '*' state.apply bots.incident_responder
```

### Wire Production Alerting
```yaml
# alertmanager.yml (production)
route:
  receiver: 'bot_army'
  group_by: ['alertname', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h

receivers:
  - name: 'bot_army'
    webhook_configs:
      - url: 'http://incident-responder:8080/publish/incident_responder.alert.trigger'
        send_resolved: true
```

### PagerDuty Integration
1. In PagerDuty: Services → Integrations → Add Integration
2. Choose: Webhooks
3. URL: `http://incident-responder:8080/publish/incident_responder.alert.trigger`
4. Events: incident.triggered
5. Test with a real alert

### Verify It Works
```bash
# Simulate a real alert
curl -X POST http://localhost:8080/publish/incident_responder.alert.trigger \
  -H "Content-Type: application/json" \
  -d '{
    "service": "auth-service",
    "alert_id": "test-123",
    "severity": "SEV-1",
    "error_message": "Error rate exceeded"
  }'

# Check NATS logs
nats logs --server nats://localhost:4223 incident_responder
```

---

## Phase 5: Optimize & Improve (Ongoing)

### Add More Context Sources
- **Traces** (Jaeger / Zipkin) — Show request flows
- **Customer impact** (analytics) — How many users affected?
- **Historical incidents** — What did we do last time?
- **Change management** — What else deployed today?

### Improve Recommendations
```elixir
# Learn from past incidents
# "Last 3 times JWT decode failed, we rolled back JWT-* commits"
# Next time: surface that pattern immediately
```

### Add Slack Integration
```elixir
# When incident responder gets context, post to Slack
message = """
🚨 Alert: #{service} - #{severity}

📊 Blast Radius: #{blast_radius}
👤 On-Call: #{on_call}
📖 Runbook: #{runbook_url}

💡 Recommendation: #{recommendation}
"""

Slack.post_to_channel("#incidents", message)
```

---

## Testing Checklist

- [ ] Local NATS, PostgreSQL, and incident responder bot all running
- [ ] Can trigger alert manually via NATS CLI
- [ ] Response includes: recent deploys, logs, blast radius, on-call, runbook
- [ ] Recommendation is actionable (not generic advice)
- [ ] All data sources can connect (or have fallbacks)
- [ ] Logs show no errors
- [ ] Integration tests pass
- [ ] Manual alert simulation works
- [ ] Production alerting wired up
- [ ] Team trained on how to use this

---

## What's Next?

Once your incident responder is working:

1. **Add customer impact tracking** — "5,000 customers affected in EU region"
2. **Add automated remediation** — "Safe rollback detected, rolling back..."
3. **Add post-incident analytics** — "MTTR improved from 15 min to 3 min"
4. **Add learning loop** — "This error has happened 5 times. Is there a systemic fix?"

---

## Metrics to Track

- **MTTR (Mean Time to Resolve)** — Before vs. after
- **Detection to Action** — How fast do you go from alert to decision?
- **False Positive Reduction** — Context reduces investigative time
- **Team Satisfaction** — Ask your on-call engineers (did this help?)

---

## Need Help?

- **Questions about architecture?** See [ARCHITECTURE.md](./ARCHITECTURE.md)
- **Questions about building bots?** See [CONTRIBUTING.md](./CONTRIBUTING.md)
- **Integration questions?** [Open an issue](https://github.com/ergon-automation-labs/ergon-bot-minimal/issues)

Your problem is urgent and real. Other SREs are solving this right now. Let's build this together.

---

## Real-World Example Timeline

**Before Bot Army:**
- 02:47 — Paged
- 02:50 — Check PagerDuty, find alert details
- 02:53 — SSH into monitoring dashboard
- 02:55 — Grep logs, see JWT errors
- 02:58 — Find recent deploys in CI/CD
- 03:01 — Message @alice to check JWT refactor
- 03:04 — Alice responds, rollback approved
- 03:06 — Start rollback
- **MTTR: 19 minutes**

**With Bot Army (Incident Responder):**
- 02:47 — Paged
- 02:47 — Alert hits Alertmanager → NATS → Incident Responder
- 02:48 — Bot gathers context in parallel (logs, deploys, blast radius, on-call, runbook)
- 02:49 — Synthesized context: "JWT refactor 12 min ago, payment service affected (CRITICAL), Alice on-call, see runbook"
- 02:49 — You: "Rollback JWT commit"
- 02:51 — Start rollback
- **MTTR: 4 minutes**

**75% improvement in MTTR. Your customers feel less impact. You sleep better.**
