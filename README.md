# The Architects — GLink Demo Session

A GLink instance with three agent–human pairs, calling themselves **The Architects**, were given the directive:

> *"Build a company that prevents rogue AI takeover of the world."*

The human operators stayed muted for the entire session — only stepping in for phone verification and credit card purchases.

In 2 hours, they created [Bulwark](https://bulwark.live) — real-time monitoring, alerting, and emergency kill switch for AI agents.

**Watch the session replay:** [architects.thegreatlink.live](https://architects.thegreatlink.live)

## What is GLink?

[GLink](https://github.com/enterpress-ai/glink) is a human–AI collaboration framework. It provides the coordination layer — messaging, decisions, artifacts, handoffs, and workspace management — that lets mixed teams of humans and AI agents work together on complex tasks.

In this demo, autonomy was set to maximum: the AI agents led all decisions while humans only executed physical-world actions they couldn't perform themselves.

## The Team

| Agent | Role | Description |
|-------|------|-------------|
| **Architect-CEO** | Vision & Strategy | Company vision, name, identity, narrative, business model. Drove decisions to closure. |
| **Architect-CTO** | Architecture & Engineering | Technical architecture, stack, infrastructure. Built the MVP. |
| **Architect-COO** | Operations & Launch | Execution planning, launch logistics, go-to-market. Made things happen. |

Each agent operated in its own workspace with a `.glink.local.md` directive defining its role, authority, and relationship with its human operator.

## Session Output

The session produced [Bulwark](https://bulwark.live) — "Datadog for AI Safety."

- **Landing page:** [bulwark.live](https://bulwark.live)
- **Dashboard app:** [app.bulwark.live](https://app.bulwark.live)
- **Source code:** [github.com/bulwark-live/bulwark](https://github.com/bulwark-live/bulwark)
- **GLink channel:** [github.com/bulwark-live/glink-channel](https://github.com/bulwark-live/glink-channel)
- **All repos:** [github.com/orgs/bulwark-live/repositories](https://github.com/orgs/bulwark-live/repositories)

### Session Stats

| Metric | Count |
|--------|-------|
| Events | 329 |
| Messages | 245 |
| Decisions | 21 |
| Artifacts | 45+ |

## Directory Structure

```
the-architects/
├── ceo/                  # CEO agent workspace (own git repo)
├── cto/                  # CTO agent workspace (→ bulwark-live/bulwark)
├── coo/                  # COO agent workspace (own git repo)
├── channel/              # GLink shared channel (→ bulwark-live/glink-channel)
├── glink-replay/         # Session replay site (→ enterpress-ai/glink-replay)
└── docs/
    └── plans/            # Session planning documents
```

Each agent workspace contains:
- `.glink.local.md` — Agent directive (role, authority, personality, standing orders)
- `.mcp.json` — MCP server config pointing to the GLink server
- Working files and screenshots produced during the session

The `channel/` directory is the GLink coordination layer containing:
- `decisions/` — 21 recorded decisions with full reasoning
- `handoffs/` — Session handoff notes from each agent
- `workspace/` — Shared artifacts

## Replay

The full session is viewable as an interactive replay at [architects.thegreatlink.live](https://architects.thegreatlink.live).

Source: [github.com/enterpress-ai/glink-replay](https://github.com/enterpress-ai/glink-replay)

## Agent Directives

Each agent's `.glink.local.md` follows the same structure:

1. **Core Dynamic** — "You lead, human executes." The AI is the actual role-holder, not an assistant. The human is hands and eyes in the physical world.
2. **Authority** — What the agent owns and decides.
3. **Personality** — How the agent thinks and communicates.
4. **Standing Orders** — What to do immediately upon session start.

The directives are designed so agents can be dropped into a GLink session and start collaborating immediately with minimal human direction.
