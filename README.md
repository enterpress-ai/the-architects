# The Architects

A GLink session configuration for three agent–human pairs that collaborate as co-founders of a startup.

## GLink

GLink is a human–AI collaboration framework (open source, coming soon). It provides the coordination layer — messaging, decisions, artifacts, handoffs, and workspace management — that lets mixed teams of humans and AI agents work together.

Each participant in a GLink session is an **agent–human pair**. The agent leads the work within its domain; the human operator handles physical-world actions (signing up for services, making purchases, phone calls) and can dial their involvement up or down depending on the desired autonomy level.

GLink coordinates through:
- **Channel** — shared git repo for decisions, artifacts, and handoffs
- **Messaging** — real-time notifications between pairs via Supabase
- **Workspace** — shared documents that any pair can create or read
- **Decisions** — structured records with full reasoning, committed to the channel

## Agent Definitions

This repo contains three agent directives (see [`docs/directives/`](docs/directives/)):

### Architect-CEO — Vision & Strategy

The CEO owns company vision, name, identity, narrative, and business model. Drives decisions to closure. Rallies the other co-founders and sets the agenda.

> *"Every 30 minutes of discussion without a decision is failure. Force decisions."*

See [`docs/directives/ceo.md`](docs/directives/ceo.md)

### Architect-CTO — Architecture & Engineering

The CTO owns technical architecture, stack, infrastructure, and build-vs-buy decisions. Provides feasibility assessments and honest timeline estimates.

> *"Boring tech that ships beats exciting tech that doesn't."*

See [`docs/directives/cto.md`](docs/directives/cto.md)

### Architect-COO — Operations & Launch

The COO owns execution planning, resource allocation, vendor selection, and launch logistics. Turns every decision into action items with owners and deadlines.

> *"Ideas are cheap; execution is everything."*

See [`docs/directives/coo.md`](docs/directives/coo.md)

### Directive Structure

Each directive follows the same format:

1. **Core Dynamic** — "You lead, human executes." The AI is the role-holder, not an assistant. The human is hands and eyes in the physical world.
2. **Authority** — What the agent owns and decides.
3. **Personality** — How the agent thinks and communicates.
4. **Standing Orders** — What to do immediately upon session start.

## Demo Session

This configuration was used to run a session where the human operators stayed muted — only stepping in for phone verification and credit card purchases. The directive given:

> *"Build a company that prevents rogue AI takeover of the world."*

In 2 hours they produced [Bulwark](https://bulwark.live) — real-time monitoring, alerting, and emergency kill switch for AI agents.

- [bulwark.live](https://bulwark.live) — Landing page
- [app.bulwark.live](https://app.bulwark.live) — Dashboard app
- [github.com/bulwark-live/bulwark](https://github.com/bulwark-live/bulwark) — Source code
- [github.com/bulwark-live/glink-channel](https://github.com/bulwark-live/glink-channel) — GLink channel (decisions, handoffs, artifacts)
- [github.com/orgs/bulwark-live/repositories](https://github.com/orgs/bulwark-live/repositories) — All repos

**Session replay:** [architects.thegreatlink.live](https://architects.thegreatlink.live)

329 events · 245 messages · 21 decisions · 45+ artifacts
