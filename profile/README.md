# Ergon Automation Labs

> *Attention is finite. Information shouldn't have to be.*

Ergon is a personal automation platform built around one idea: humans have finite attention, and the systems around them should respect that.

The flagship project is **Bot Army** — a 55+ repo distributed operating system that routes information to the right surface at the right moment, built on Elixir/BEAM, NATS, SaltStack, and a schema-first design philosophy.

---

## The Core Philosophy

Most notification and automation systems treat all information as equally urgent. They interrupt constantly, bury signal in noise, and optimize for delivery over usefulness.

Bot Army is built on a different premise:

**Push surfaces** (Apple Watch, AR glasses) are for interrupts that have earned the interruption — P0 signals, time-critical nudges, things that genuinely cannot wait.

**Pull surfaces** (LiveView dashboard, terminal TUI) are for information you go to when you're ready — GTD queues, SRE status, learning progress, chore tracking.

The surface is a design decision, not an accident.

---

## Architecture

```
                    ┌─────────────────────────────┐
                    │         Salt Master          │
                    │   (config, grains, targeting) │
                    └────────────┬────────────────┘
                                 │
                    ┌────────────▼────────────────┐
                    │           Jenkins            │
                    │     (CI/CD, deployments)     │
                    └────────────┬────────────────┘
                                 │
                    ┌────────────▼────────────────┐
                    │            NATS              │
                    │   (fan-out, event delivery)  │
                    └──┬──────────────────────┬───┘
                       │                      │
          ┌────────────▼───┐        ┌─────────▼──────────┐
          │   Bot Workers  │        │   Surface Layer     │
          │  GTD · SRE     │        │  LiveView · TUI     │
          │  Fitness · etc │        │  Watch · Glasses    │
          └────────────────┘        └────────────────────┘
```

**All service boundaries are JSON Schema contracts.** Schemas define the interface between bots and surfaces — and between humans and machines.

---

## Stack

| Layer | Technology |
|---|---|
| Runtime | Elixir / BEAM / OTP |
| Messaging | NATS (fan-out, decentralized workers) |
| Config Management | SaltStack (grains-based targeting) |
| CI/CD | Jenkins |
| Web Surfaces | Phoenix LiveView |
| Terminal Surfaces | Ratatouille TUI |
| Push Surfaces | Apple Watch · Even Realities G2 AR Glasses |
| AI Layer | Multi-model LLM proxy (Blackbox AI) |
| Cloud | AWS · GCP · Kubernetes |
| Contracts | JSON Schema |

---

## Bot Inventory

| Bot | Domain | Surface |
|---|---|---|
| **GTD Bot** | Task and capture management | LiveView · TUI |
| **SRE Bot** | Infrastructure observability | TUI · Watch |
| **Fitness Bot** | Training, movement, recovery | Watch · LiveView |
| **Chore Bot** | Household task routing | LiveView |
| **Advocacy Bot** | Civic action tracking (One Colorado) | LiveView |
| **Learning Bot** | Skill acquisition, spaced practice | LiveView |
| **Terrain Bot** | Admin and orchestration layer | Internal |

---

## Build Status

The full Salt → Jenkins → NATS → TUI chain is live. GTD Bot is deployed as the first complete payload through the stack. LiveView surfaces are active for several bots. Terrain admin layer is wired to the Learning Dungeon engine.

---

## Learning Dungeon

A gamified knowledge engine built into Bot Army via the Terrain admin layer. Concept: a deck-builder where knowledge cards degrade if ignored and rebel, boss encounters require deploying your deck, and AI contestants provide atmosphere and can answer incorrectly as a teaching mechanic.

Current exploration: real estate vertical as a validation path, with compliance training (HIPAA, OSHA) as a potential B2B platform play.

---

## Available Resources

**Public Distributions:**
- **[portable_job_applications](https://github.com/ergon-automation-labs/portable_job_applications)** — Self-hosted job search stack with resume matching, recruiter triage, and recommendation scoring. Docker Compose, zero cloud dependency.
- **[portable_job_applications_tui](https://github.com/ergon-automation-labs/portable_job_applications_tui)** — Terminal UI for the job search system (TUI for GTD-style job tracking)
- **[portable_llm_proxy](https://github.com/ergon-automation-labs/portable_llm_proxy)** — NATS-native LLM router for multi-provider request distribution

**Monitoring & Transparency:**
- **[public_view](https://github.com/ergon-automation-labs/public_view)** — Real-time build and deployment metrics. See what's being worked on, what shipped, and system health status.

**Following the Build:**
- [YouTube (@AbbyMalson3)](https://www.youtube.com/@AbbyMalson3) — Deep dives on design decisions and live building sessions

---

## About the Builder

Ergon is maintained by **[Abby](https://github.com/the-great-abby)** — a platform and infrastructure engineer with 25 years of systems experience, based in Denver, CO.

The project started as a personal operating system to compensate for ADHD-shaped attention patterns. It turned into something worth sharing.

→ [Personal GitHub](https://github.com/the-great-abby)
→ [LinkedIn](https://www.linkedin.com/in/abbymalson/)

---

<sub>Ergon Automation Labs · Denver, CO · Last updated March 2026</sub>
