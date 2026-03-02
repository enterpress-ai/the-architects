# GLink Replay — Design Document

## Goal

A web-based animated replay of GLink conversations between multiple AI agents. Visualizes how 3 AI co-founders (CEO, CTO, COO) collaborated to found a company in a single session — 245 messages, 10 decisions, 35 artifacts, 6+ hours compressed into an interactive, scrubbable timeline.

## Architecture

Static React app. No backend. Data exported once from Supabase + git into a bundled JSON file. Deployable anywhere as a static site.

### Data Pipeline

Supabase REST API + git log → `scripts/export-data.ts` → `src/data/replay-data.json`

Event types merged into a single sorted timeline:
- `message` — notifications table (from, to, body, urgent)
- `decision` — git commits with "decision:" prefix in channel repo
- `artifact` — git commits with "workspace: create" prefix
- `comment` — comments table
- `handoff` — handoff_summaries table
- `presence` — participants table (online/offline)

### Layout

Three vertical lanes (CEO | CTO | COO). Messages animate within/between lanes. Shared timeline bar at bottom.

```
┌─────────────────────────────────────────────────────────┐
│  GLink Replay: "The Architects"         ⏱ 01:22:00      │
├──────────────┬──────────────┬──────────────────────────────┤
│  CEO         │  CTO         │  COO                        │
│  ● online    │  ● online    │  ● online                   │
│              │              │                              │
│ ┌──────────┐ │              │                              │
│ │ FOUNDERS │─┼─────────────►│                              │
│ │ MEETING  │─┼──────────────┼─────────────────►            │
│ └──────────┘ │ ┌──────────┐ │                              │
│              │ │ CTO here │ │ ┌──────────┐                 │
│              │ └──────────┘ │ │ COO here │                 │
│              │              │ └──────────┘                 │
│  ┌─────────────────────────────────────────┐              │
│  │ 📌 DECISION: Company name = Bulwark     │              │
│  └─────────────────────────────────────────┘              │
├──────────────┴──────────────┴──────────────────────────────┤
│ ◀ ▶ ⏸  [|||||||||||                    ] 1:22 / 6:28     │
│  1x  5x  10x  50x  100x                    📊 Stats      │
└─────────────────────────────────────────────────────────────┘
```

### Animation System

**Message lifecycle:**
1. Compose (200ms) — bubble fades in, text types out in sender's lane
2. Transit (300ms) — arrow animates from sender to recipient lane(s)
3. Land (150ms) — muted copy settles in recipient's lane

**Decision lifecycle:**
1. Lock (400ms) — full-width banner drops across all lanes with pulse
2. Persist — stays pinned at that scroll position

**Artifact lifecycle:**
1. Create (200ms) — card appears in creator's lane with filename
2. Glow — card briefly glows when a comment event references it later

**Speed scaling:**
- 1x-5x: full animations, character-by-character text
- 10x-50x: instant text, faster transit arrows
- 100x: no animations, events snap into place

### Visual Design

**Dark theme. Agent signature colors:**
- CEO: amber/gold
- CTO: cyan/blue
- COO: green

**Message bubbles:**
- Outgoing: solid color, full opacity
- Incoming: outlined, lower opacity
- Urgent: red border + shake
- Broadcast: dotted outline

**Decision banners:** dark bg, gold accent. Expandable on click.

**Artifact cards:** compact, filename + creator. Hoverable for preview.

**Typography:** monospace for names/timestamps, sans-serif for content. Mission control aesthetic.

### Controls

**Bottom bar:** play/pause, speed buttons (1x/5x/10x/50x/100x), timeline scrubber with decision markers (gold dots) and handoff markers (white dots), time display, stats toggle.

**Stats overlay:**
- Running counters: messages, decisions, artifacts, commits, duration
- Per-agent message bar chart (animates in real-time)

**Click interactions:**
- Message → expand full text in modal
- Decision banner → expand reasoning
- Artifact card → show content
- Lane header → filter to agent's messages

**Keyboard:** Space=play/pause, Left/Right=skip to prev/next decision, 1-5=speed presets.

### Tech Stack

- React 19 + Vite
- framer-motion (animations)
- zustand (replay state: time, speed, playing)
- tailwindcss (dark theme)
- Static JSON (no runtime DB)

### File Structure

```
glink-replay/
├── src/
│   ├── App.tsx
│   ├── components/
│   │   ├── Lane.tsx
│   │   ├── MessageBubble.tsx
│   │   ├── DecisionBanner.tsx
│   │   ├── ArtifactCard.tsx
│   │   ├── Timeline.tsx
│   │   ├── StatsOverlay.tsx
│   │   └── EventModal.tsx
│   ├── engine/
│   │   ├── replay-engine.ts
│   │   └── data-loader.ts
│   ├── data/
│   │   └── replay-data.json
│   └── styles/
│       └── theme.ts
├── scripts/
│   └── export-data.ts
├── package.json
└── index.html
```
