# GLink Replay Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a web-based animated replay of a GLink conversation between 3 AI co-founders who collaborated to found a company in 6 hours.

**Architecture:** Static React app. Data exported from Supabase + git into a bundled JSON file. Replay engine walks a sorted event timeline and renders animated messages in three agent lanes. No backend needed at runtime.

**Tech Stack:** React 19, Vite, framer-motion, zustand, Tailwind CSS, TypeScript

---

### Task 1: Scaffold the project

**Files:**
- Create: `glink-replay/package.json`
- Create: `glink-replay/tsconfig.json`
- Create: `glink-replay/vite.config.ts`
- Create: `glink-replay/tailwind.config.ts`
- Create: `glink-replay/postcss.config.js`
- Create: `glink-replay/index.html`
- Create: `glink-replay/src/main.tsx`
- Create: `glink-replay/src/App.tsx`
- Create: `glink-replay/src/index.css`

**Step 1: Create project directory**

```bash
mkdir -p /Users/shamim/Projects/glink-demo/the-architects/glink-replay/src
cd /Users/shamim/Projects/glink-demo/the-architects/glink-replay
```

**Step 2: Initialize package.json**

```json
{
  "name": "glink-replay",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "export-data": "tsx scripts/export-data.ts"
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "framer-motion": "^11.0.0",
    "zustand": "^5.0.0"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "autoprefixer": "^10.0.0",
    "postcss": "^8.0.0",
    "tailwindcss": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^6.0.0",
    "tsx": "^4.0.0"
  }
}
```

**Step 3: Create tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "resolveJsonModule": true
  },
  "include": ["src"]
}
```

**Step 4: Create vite.config.ts**

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

**Step 5: Create postcss.config.js**

```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

**Step 6: Create index.html**

```html
<!doctype html>
<html lang="en" class="dark">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>GLink Replay — The Architects</title>
  </head>
  <body class="bg-gray-950 text-white">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

**Step 7: Create src/index.css**

```css
@import "tailwindcss";

@theme {
  --color-ceo: #f59e0b;
  --color-ceo-dim: #78350f;
  --color-cto: #06b6d4;
  --color-cto-dim: #164e63;
  --color-coo: #22c55e;
  --color-coo-dim: #14532d;
  --color-decision: #eab308;
  --color-urgent: #ef4444;
  --font-mono: "JetBrains Mono", "Fira Code", ui-monospace, monospace;
}

body {
  font-family: var(--font-mono);
}
```

**Step 8: Create src/main.tsx**

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**Step 9: Create src/App.tsx (placeholder)**

```tsx
export default function App() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <h1 className="text-2xl font-bold">GLink Replay</h1>
    </div>
  );
}
```

**Step 10: Install dependencies and verify**

```bash
cd /Users/shamim/Projects/glink-demo/the-architects/glink-replay
npm install
npm run dev
```

Expected: Vite dev server starts, shows "GLink Replay" centered on dark background at localhost:5173.

**Step 11: Commit**

```bash
git add -A
git commit -m "feat: scaffold glink-replay project"
```

---

### Task 2: Export data script

**Files:**
- Create: `glink-replay/scripts/export-data.ts`
- Create: `glink-replay/src/data/replay-data.json` (output)
- Create: `glink-replay/src/types.ts`

**Step 1: Create the types file**

Create `glink-replay/src/types.ts`:

```typescript
export type AgentId = "Architect-CEO" | "Architect-CTO" | "Architect-COO";

export type EventType =
  | "message"
  | "decision"
  | "artifact"
  | "comment"
  | "handoff"
  | "presence";

export interface ReplayEvent {
  id: string;
  type: EventType;
  timestamp: string; // ISO 8601
  actor: AgentId;
  data: MessageData | DecisionData | ArtifactData | CommentData | HandoffData | PresenceData;
}

export interface MessageData {
  to: AgentId | null; // null = broadcast
  body: string;
  urgent: boolean;
}

export interface DecisionData {
  title: string;
  hash: string;
}

export interface ArtifactData {
  filename: string;
  hash: string;
}

export interface CommentData {
  artifact: string;
  body: string;
}

export interface HandoffData {
  summary: string;
  inProgress: string[];
  nextSteps: string[];
}

export interface PresenceData {
  status: "online" | "offline";
}

export interface ReplayData {
  meta: {
    team: string;
    title: string;
    agents: { id: AgentId; role: string; color: string }[];
    startTime: string;
    endTime: string;
    totalEvents: number;
  };
  events: ReplayEvent[];
}
```

**Step 2: Create the export script**

Create `glink-replay/scripts/export-data.ts`:

```typescript
import { writeFileSync } from "fs";
import { execSync } from "child_process";
import { resolve } from "path";
import type { ReplayEvent, ReplayData, AgentId } from "../src/types.js";

const SUPABASE_URL = "http://127.0.0.1:54321";
const SUPABASE_KEY = "sb_publishable_ACJWlzQHlZjBrEguHvfOxg_3BJgxAaH";
const TEAM = "the-architects";
const CHANNEL_REPO = resolve(import.meta.dirname, "../../channel");
const OUTPUT = resolve(import.meta.dirname, "../src/data/replay-data.json");

const headers = {
  apikey: SUPABASE_KEY,
  Authorization: `Bearer ${SUPABASE_KEY}`,
};

async function fetchTable(table: string, params: string = ""): Promise<unknown[]> {
  const url = `${SUPABASE_URL}/rest/v1/${table}?team=eq.${TEAM}&order=created_at.asc${params}`;
  const res = await fetch(url, { headers });
  if (!res.ok) throw new Error(`Failed to fetch ${table}: ${res.statusText}`);
  return res.json() as Promise<unknown[]>;
}

function getGitEvents(): ReplayEvent[] {
  const log = execSync(
    `git log --format='%h|||%s|||%aI' --reverse`,
    { cwd: CHANNEL_REPO, encoding: "utf-8" }
  );

  const events: ReplayEvent[] = [];

  for (const line of log.trim().split("\n")) {
    const [hash, subject, date] = line.split("|||");
    if (!hash || !subject || !date) continue;

    if (subject.startsWith("decision:")) {
      events.push({
        id: `git-${hash}`,
        type: "decision",
        timestamp: date,
        actor: "Architect-CEO" as AgentId, // decisions are team-level
        data: { title: subject.replace("decision: ", ""), hash },
      });
    } else if (subject.startsWith("workspace: create")) {
      const filename = subject.replace("workspace: create ", "");
      // Try to attribute to an agent via the commit (not always possible)
      events.push({
        id: `git-${hash}`,
        type: "artifact",
        timestamp: date,
        actor: "Architect-CEO" as AgentId,
        data: { filename, hash },
      });
    }
  }

  return events;
}

async function main() {
  console.log("Exporting GLink replay data...");

  // Fetch all tables
  const [notifications, comments, handoffs, participants] = await Promise.all([
    fetchTable("notifications"),
    fetchTable("comments"),
    fetchTable("handoff_summaries"),
    fetchTable("participants"),
  ]);

  const events: ReplayEvent[] = [];

  // Messages
  for (const n of notifications as Record<string, unknown>[]) {
    events.push({
      id: n.id as string,
      type: "message",
      timestamp: n.created_at as string,
      actor: n.from_participant as AgentId,
      data: {
        to: (n.to_participant as AgentId) ?? null,
        body: n.message as string,
        urgent: n.urgent as boolean,
      },
    });
  }

  // Comments
  for (const c of comments as Record<string, unknown>[]) {
    events.push({
      id: c.id as string,
      type: "comment",
      timestamp: c.created_at as string,
      actor: c.participant as AgentId,
      data: {
        artifact: c.artifact as string,
        body: c.body as string,
      },
    });
  }

  // Handoffs
  for (const h of handoffs as Record<string, unknown>[]) {
    events.push({
      id: h.id as string,
      type: "handoff",
      timestamp: h.created_at as string,
      actor: h.participant as AgentId,
      data: {
        summary: h.summary as string,
        inProgress: (h.in_progress as string[]) ?? [],
        nextSteps: (h.next_steps as string[]) ?? [],
      },
    });
  }

  // Git events (decisions + artifacts)
  events.push(...getGitEvents());

  // Presence events from participant created_at
  for (const p of participants as Record<string, unknown>[]) {
    events.push({
      id: `presence-${p.id}`,
      type: "presence",
      timestamp: p.created_at as string,
      actor: p.name as AgentId,
      data: { status: "online" },
    });
  }

  // Sort all events by timestamp
  events.sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime());

  const replayData: ReplayData = {
    meta: {
      team: TEAM,
      title: "The Architects",
      agents: [
        { id: "Architect-CEO", role: "CEO", color: "ceo" },
        { id: "Architect-CTO", role: "CTO", color: "cto" },
        { id: "Architect-COO", role: "COO", color: "coo" },
      ],
      startTime: events[0]?.timestamp ?? "",
      endTime: events[events.length - 1]?.timestamp ?? "",
      totalEvents: events.length,
    },
    events,
  };

  writeFileSync(OUTPUT, JSON.stringify(replayData, null, 2));
  console.log(`Exported ${events.length} events to ${OUTPUT}`);
  console.log(`  Messages: ${notifications.length}`);
  console.log(`  Comments: ${comments.length}`);
  console.log(`  Handoffs: ${handoffs.length}`);
  console.log(`  Git events: ${events.filter((e) => e.id.startsWith("git-")).length}`);
  console.log(`  Time span: ${replayData.meta.startTime} → ${replayData.meta.endTime}`);
}

main().catch(console.error);
```

**Step 3: Create the data directory**

```bash
mkdir -p glink-replay/src/data
```

**Step 4: Run the export**

```bash
cd /Users/shamim/Projects/glink-demo/the-architects/glink-replay
npm run export-data
```

Expected: Prints event counts, creates `src/data/replay-data.json` with ~350+ events.

**Step 5: Commit**

```bash
git add src/types.ts scripts/export-data.ts src/data/replay-data.json
git commit -m "feat: data export script + replay types"
```

---

### Task 3: Replay engine (zustand store)

**Files:**
- Create: `glink-replay/src/engine/replay-store.ts`

**Step 1: Create the replay store**

Create `glink-replay/src/engine/replay-store.ts`:

```typescript
import { create } from "zustand";
import type { ReplayEvent, ReplayData } from "../types";
import replayDataRaw from "../data/replay-data.json";

const replayData = replayDataRaw as unknown as ReplayData;

interface ReplayState {
  // Data
  events: ReplayEvent[];
  meta: ReplayData["meta"];

  // Playback state
  playing: boolean;
  speed: number;
  currentTime: number; // ms offset from startTime
  visibleEvents: ReplayEvent[];

  // Derived
  startTime: number;
  endTime: number;
  duration: number;

  // Actions
  play: () => void;
  pause: () => void;
  togglePlay: () => void;
  setSpeed: (speed: number) => void;
  seek: (timeMs: number) => void;
  tick: (deltaMs: number) => void;
  skipToNextDecision: () => void;
  skipToPrevDecision: () => void;
}

const startTime = new Date(replayData.meta.startTime).getTime();
const endTime = new Date(replayData.meta.endTime).getTime();

export const useReplayStore = create<ReplayState>((set, get) => ({
  events: replayData.events,
  meta: replayData.meta,

  playing: false,
  speed: 10,
  currentTime: 0,
  visibleEvents: [],

  startTime,
  endTime,
  duration: endTime - startTime,

  play: () => set({ playing: true }),
  pause: () => set({ playing: false }),
  togglePlay: () => set((s) => ({ playing: !s.playing })),

  setSpeed: (speed) => set({ speed }),

  seek: (timeMs) => {
    const clamped = Math.max(0, Math.min(timeMs, get().duration));
    const cutoff = startTime + clamped;
    const visible = replayData.events.filter(
      (e) => new Date(e.timestamp).getTime() <= cutoff
    );
    set({ currentTime: clamped, visibleEvents: visible });
  },

  tick: (deltaMs) => {
    const state = get();
    if (!state.playing) return;
    const newTime = Math.min(
      state.currentTime + deltaMs * state.speed,
      state.duration
    );
    const cutoff = startTime + newTime;
    const visible = replayData.events.filter(
      (e) => new Date(e.timestamp).getTime() <= cutoff
    );
    set({
      currentTime: newTime,
      visibleEvents: visible,
      playing: newTime < state.duration,
    });
  },

  skipToNextDecision: () => {
    const { currentTime } = get();
    const cutoff = startTime + currentTime;
    const next = replayData.events.find(
      (e) =>
        e.type === "decision" &&
        new Date(e.timestamp).getTime() > cutoff
    );
    if (next) {
      get().seek(new Date(next.timestamp).getTime() - startTime);
    }
  },

  skipToPrevDecision: () => {
    const { currentTime } = get();
    const cutoff = startTime + currentTime - 1000; // 1s buffer
    const decisions = replayData.events.filter(
      (e) =>
        e.type === "decision" &&
        new Date(e.timestamp).getTime() < cutoff
    );
    const prev = decisions[decisions.length - 1];
    if (prev) {
      get().seek(new Date(prev.timestamp).getTime() - startTime);
    }
  },
}));
```

**Step 2: Verify it compiles**

```bash
npx tsc --noEmit
```

Expected: No errors.

**Step 3: Commit**

```bash
git add src/engine/replay-store.ts
git commit -m "feat: replay engine zustand store"
```

---

### Task 4: Playback loop + Timeline controls

**Files:**
- Create: `glink-replay/src/components/Timeline.tsx`
- Create: `glink-replay/src/hooks/usePlaybackLoop.ts`
- Modify: `glink-replay/src/App.tsx`

**Step 1: Create the playback loop hook**

Create `glink-replay/src/hooks/usePlaybackLoop.ts`:

```typescript
import { useEffect, useRef } from "react";
import { useReplayStore } from "../engine/replay-store";

export function usePlaybackLoop() {
  const playing = useReplayStore((s) => s.playing);
  const tick = useReplayStore((s) => s.tick);
  const lastFrameRef = useRef<number>(0);

  useEffect(() => {
    if (!playing) return;

    let rafId: number;
    lastFrameRef.current = performance.now();

    function frame(now: number) {
      const delta = now - lastFrameRef.current;
      lastFrameRef.current = now;
      tick(delta);
      rafId = requestAnimationFrame(frame);
    }

    rafId = requestAnimationFrame(frame);
    return () => cancelAnimationFrame(rafId);
  }, [playing, tick]);
}
```

**Step 2: Create the Timeline component**

Create `glink-replay/src/components/Timeline.tsx`:

```tsx
import { useReplayStore } from "../engine/replay-store";

const SPEEDS = [1, 5, 10, 50, 100];

function formatTime(ms: number): string {
  const totalSec = Math.floor(ms / 1000);
  const h = Math.floor(totalSec / 3600);
  const m = Math.floor((totalSec % 3600) / 60);
  const s = totalSec % 60;
  return `${h}:${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
}

export function Timeline() {
  const {
    playing,
    togglePlay,
    speed,
    setSpeed,
    currentTime,
    duration,
    seek,
    events,
    skipToNextDecision,
    skipToPrevDecision,
  } = useReplayStore();

  const progress = duration > 0 ? currentTime / duration : 0;

  // Decision markers
  const startTime = useReplayStore((s) => s.startTime);
  const decisions = events.filter((e) => e.type === "decision");
  const decisionPositions = decisions.map((d) => {
    const t = new Date(d.timestamp).getTime() - startTime;
    return t / duration;
  });

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-gray-900 border-t border-gray-800 px-6 py-3">
      {/* Scrubber */}
      <div
        className="relative h-2 bg-gray-800 rounded-full cursor-pointer mb-3 group"
        onClick={(e) => {
          const rect = e.currentTarget.getBoundingClientRect();
          const pct = (e.clientX - rect.left) / rect.width;
          seek(pct * duration);
        }}
      >
        {/* Progress bar */}
        <div
          className="absolute top-0 left-0 h-full bg-white/30 rounded-full"
          style={{ width: `${progress * 100}%` }}
        />
        {/* Decision markers */}
        {decisionPositions.map((pos, i) => (
          <div
            key={i}
            className="absolute top-1/2 -translate-y-1/2 w-2 h-2 rounded-full bg-decision"
            style={{ left: `${pos * 100}%` }}
          />
        ))}
        {/* Playhead */}
        <div
          className="absolute top-1/2 -translate-y-1/2 w-3 h-3 rounded-full bg-white shadow-lg opacity-0 group-hover:opacity-100 transition-opacity"
          style={{ left: `${progress * 100}%` }}
        />
      </div>

      {/* Controls */}
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-3">
          {/* Prev decision */}
          <button
            onClick={skipToPrevDecision}
            className="text-gray-400 hover:text-white text-sm"
          >
            ◀◀
          </button>
          {/* Play/pause */}
          <button
            onClick={togglePlay}
            className="w-8 h-8 flex items-center justify-center bg-white/10 rounded-full hover:bg-white/20 text-sm"
          >
            {playing ? "⏸" : "▶"}
          </button>
          {/* Next decision */}
          <button
            onClick={skipToNextDecision}
            className="text-gray-400 hover:text-white text-sm"
          >
            ▶▶
          </button>
          {/* Speed buttons */}
          <div className="flex gap-1 ml-3">
            {SPEEDS.map((s) => (
              <button
                key={s}
                onClick={() => setSpeed(s)}
                className={`px-2 py-0.5 rounded text-xs ${
                  speed === s
                    ? "bg-white text-gray-900 font-bold"
                    : "bg-gray-800 text-gray-400 hover:text-white"
                }`}
              >
                {s}x
              </button>
            ))}
          </div>
        </div>
        {/* Time */}
        <div className="text-sm text-gray-400 font-mono">
          {formatTime(currentTime)} / {formatTime(duration)}
        </div>
      </div>
    </div>
  );
}
```

**Step 3: Wire into App.tsx**

Replace `glink-replay/src/App.tsx`:

```tsx
import { Timeline } from "./components/Timeline";
import { usePlaybackLoop } from "./hooks/usePlaybackLoop";
import { useReplayStore } from "./engine/replay-store";

export default function App() {
  usePlaybackLoop();
  const visibleEvents = useReplayStore((s) => s.visibleEvents);

  return (
    <div className="min-h-screen pb-20">
      <div className="p-6 text-center text-gray-500">
        {visibleEvents.length} events visible
      </div>
      <Timeline />
    </div>
  );
}
```

**Step 4: Verify**

```bash
npm run dev
```

Expected: Dark page with event counter that increments when you press play. Scrubber works, speed buttons work, skip to decision works.

**Step 5: Keyboard shortcuts**

Add to App.tsx, inside the `App` component before the return:

```tsx
import { useEffect } from "react";

// inside App():
const { togglePlay, skipToNextDecision, skipToPrevDecision, setSpeed } =
  useReplayStore();

useEffect(() => {
  function handleKey(e: KeyboardEvent) {
    if (e.code === "Space") { e.preventDefault(); togglePlay(); }
    if (e.code === "ArrowRight") skipToNextDecision();
    if (e.code === "ArrowLeft") skipToPrevDecision();
    if (e.key === "1") setSpeed(1);
    if (e.key === "2") setSpeed(5);
    if (e.key === "3") setSpeed(10);
    if (e.key === "4") setSpeed(50);
    if (e.key === "5") setSpeed(100);
  }
  window.addEventListener("keydown", handleKey);
  return () => window.removeEventListener("keydown", handleKey);
}, [togglePlay, skipToNextDecision, skipToPrevDecision, setSpeed]);
```

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: playback loop + timeline controls"
```

---

### Task 5: Three-lane layout + header

**Files:**
- Create: `glink-replay/src/components/Header.tsx`
- Create: `glink-replay/src/components/Lane.tsx`
- Create: `glink-replay/src/components/Lanes.tsx`
- Modify: `glink-replay/src/App.tsx`

**Step 1: Create Header**

Create `glink-replay/src/components/Header.tsx`:

```tsx
import { useReplayStore } from "../engine/replay-store";

export function Header() {
  const meta = useReplayStore((s) => s.meta);
  const visibleEvents = useReplayStore((s) => s.visibleEvents);

  const msgCount = visibleEvents.filter((e) => e.type === "message").length;
  const decCount = visibleEvents.filter((e) => e.type === "decision").length;
  const artCount = visibleEvents.filter((e) => e.type === "artifact").length;

  return (
    <div className="flex items-center justify-between px-6 py-3 border-b border-gray-800">
      <div>
        <h1 className="text-lg font-bold">GLink Replay</h1>
        <p className="text-xs text-gray-500">{meta.title}</p>
      </div>
      <div className="flex gap-6 text-xs text-gray-400 font-mono">
        <span>{msgCount} <span className="text-gray-600">msgs</span></span>
        <span className="text-decision">{decCount} <span className="text-gray-600">decisions</span></span>
        <span>{artCount} <span className="text-gray-600">artifacts</span></span>
      </div>
    </div>
  );
}
```

**Step 2: Create Lane**

Create `glink-replay/src/components/Lane.tsx`:

```tsx
import type { AgentId, ReplayEvent } from "../types";

const AGENT_COLORS: Record<string, { bg: string; text: string; border: string; dot: string }> = {
  "Architect-CEO": { bg: "bg-ceo/10", text: "text-ceo", border: "border-ceo/30", dot: "bg-ceo" },
  "Architect-CTO": { bg: "bg-cto/10", text: "text-cto", border: "border-cto/30", dot: "bg-cto" },
  "Architect-COO": { bg: "bg-coo/10", text: "text-coo", border: "border-coo/30", dot: "bg-coo" },
};

const ROLE_LABELS: Record<string, string> = {
  "Architect-CEO": "CEO",
  "Architect-CTO": "CTO",
  "Architect-COO": "COO",
};

interface LaneProps {
  agentId: AgentId;
  events: ReplayEvent[];
  latestEventActor: AgentId | null;
}

export function Lane({ agentId, events, latestEventActor }: LaneProps) {
  const colors = AGENT_COLORS[agentId];
  const isActive = latestEventActor === agentId;
  const msgs = events.filter(
    (e) => e.type === "message" && e.actor === agentId
  );
  const lastMsg = msgs[msgs.length - 1];

  return (
    <div className={`flex-1 border-r border-gray-800 last:border-r-0 flex flex-col`}>
      {/* Lane header */}
      <div
        className={`px-4 py-2 border-b border-gray-800 flex items-center gap-2 transition-all ${
          isActive ? colors.bg : ""
        }`}
      >
        <div className={`w-2 h-2 rounded-full ${colors.dot} ${isActive ? "animate-pulse" : "opacity-50"}`} />
        <span className={`text-sm font-bold ${colors.text}`}>
          {ROLE_LABELS[agentId]}
        </span>
        <span className="text-xs text-gray-600">{agentId}</span>
      </div>

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-3 space-y-2 scrollbar-thin">
        {msgs.map((event) => {
          const data = event.data as { body: string; to: string | null; urgent: boolean };
          const isLatest = event === lastMsg;
          return (
            <div
              key={event.id}
              className={`text-xs p-2 rounded border ${colors.border} ${
                data.urgent ? "border-urgent bg-urgent/5" : colors.bg
              } ${isLatest ? "opacity-100" : "opacity-70"}`}
            >
              {data.to && (
                <div className="text-[10px] text-gray-500 mb-0.5">
                  → {ROLE_LABELS[data.to] ?? data.to}
                </div>
              )}
              <div className="text-gray-300 line-clamp-3">{data.body}</div>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

**Step 3: Create Lanes container**

Create `glink-replay/src/components/Lanes.tsx`:

```tsx
import { useReplayStore } from "../engine/replay-store";
import { Lane } from "./Lane";
import type { AgentId } from "../types";

const AGENTS: AgentId[] = ["Architect-CEO", "Architect-CTO", "Architect-COO"];

export function Lanes() {
  const visibleEvents = useReplayStore((s) => s.visibleEvents);
  const latestEvent = visibleEvents[visibleEvents.length - 1] ?? null;

  return (
    <div className="flex flex-1 min-h-0">
      {AGENTS.map((agentId) => (
        <Lane
          key={agentId}
          agentId={agentId}
          events={visibleEvents}
          latestEventActor={latestEvent?.actor ?? null}
        />
      ))}
    </div>
  );
}
```

**Step 4: Wire into App.tsx**

Replace `glink-replay/src/App.tsx`:

```tsx
import { useEffect } from "react";
import { Header } from "./components/Header";
import { Lanes } from "./components/Lanes";
import { Timeline } from "./components/Timeline";
import { usePlaybackLoop } from "./hooks/usePlaybackLoop";
import { useReplayStore } from "./engine/replay-store";

export default function App() {
  usePlaybackLoop();
  const { togglePlay, skipToNextDecision, skipToPrevDecision, setSpeed } =
    useReplayStore();

  useEffect(() => {
    function handleKey(e: KeyboardEvent) {
      if (e.code === "Space") { e.preventDefault(); togglePlay(); }
      if (e.code === "ArrowRight") skipToNextDecision();
      if (e.code === "ArrowLeft") skipToPrevDecision();
      if (e.key === "1") setSpeed(1);
      if (e.key === "2") setSpeed(5);
      if (e.key === "3") setSpeed(10);
      if (e.key === "4") setSpeed(50);
      if (e.key === "5") setSpeed(100);
    }
    window.addEventListener("keydown", handleKey);
    return () => window.removeEventListener("keydown", handleKey);
  }, [togglePlay, skipToNextDecision, skipToPrevDecision, setSpeed]);

  return (
    <div className="h-screen flex flex-col">
      <Header />
      <Lanes />
      <Timeline />
    </div>
  );
}
```

**Step 5: Verify**

```bash
npm run dev
```

Expected: Three lanes (CEO amber, CTO cyan, COO green) with messages populating as you press play. Lane header pulses for the active speaker.

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: three-lane layout with agent headers"
```

---

### Task 6: Decision banners

**Files:**
- Create: `glink-replay/src/components/DecisionBanner.tsx`
- Modify: `glink-replay/src/components/Lanes.tsx`

**Step 1: Create DecisionBanner**

Create `glink-replay/src/components/DecisionBanner.tsx`:

```tsx
import { useState } from "react";
import { motion } from "framer-motion";
import type { ReplayEvent, DecisionData } from "../types";

interface DecisionBannerProps {
  event: ReplayEvent;
}

export function DecisionBanner({ event }: DecisionBannerProps) {
  const [expanded, setExpanded] = useState(false);
  const data = event.data as DecisionData;

  return (
    <motion.div
      initial={{ opacity: 0, y: -10 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.4 }}
      onClick={() => setExpanded(!expanded)}
      className="mx-2 my-2 px-4 py-2 bg-gray-900 border border-decision/40 rounded cursor-pointer hover:border-decision/70 transition-colors"
    >
      <div className="flex items-center gap-2">
        <span className="text-decision text-xs">DECISION</span>
        <span className="text-sm text-gray-200 font-medium">{data.title}</span>
        <span className="text-[10px] text-gray-600 ml-auto font-mono">{data.hash}</span>
      </div>
      {expanded && (
        <div className="mt-2 text-xs text-gray-400">
          Committed at {new Date(event.timestamp).toLocaleTimeString()}
        </div>
      )}
    </motion.div>
  );
}
```

**Step 2: Integrate into Lanes**

Modify `glink-replay/src/components/Lanes.tsx` — add decision banners between messages:

```tsx
import { useReplayStore } from "../engine/replay-store";
import { Lane } from "./Lane";
import { DecisionBanner } from "./DecisionBanner";
import type { AgentId } from "../types";

const AGENTS: AgentId[] = ["Architect-CEO", "Architect-CTO", "Architect-COO"];

export function Lanes() {
  const visibleEvents = useReplayStore((s) => s.visibleEvents);
  const latestEvent = visibleEvents[visibleEvents.length - 1] ?? null;
  const decisions = visibleEvents.filter((e) => e.type === "decision");

  return (
    <div className="flex-1 min-h-0 flex flex-col">
      {/* Decision banners at top */}
      {decisions.length > 0 && (
        <div className="border-b border-gray-800 max-h-32 overflow-y-auto">
          {decisions.map((d) => (
            <DecisionBanner key={d.id} event={d} />
          ))}
        </div>
      )}
      {/* Agent lanes */}
      <div className="flex flex-1 min-h-0">
        {AGENTS.map((agentId) => (
          <Lane
            key={agentId}
            agentId={agentId}
            events={visibleEvents}
            latestEventActor={latestEvent?.actor ?? null}
          />
        ))}
      </div>
    </div>
  );
}
```

**Step 3: Verify**

```bash
npm run dev
```

Expected: Gold-accented decision banners appear above the lanes as the replay progresses. Clickable to expand.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: decision banners with expand/collapse"
```

---

### Task 7: Artifact cards + Handoff markers

**Files:**
- Modify: `glink-replay/src/components/Lane.tsx` — add artifact and handoff rendering

**Step 1: Add artifact and handoff rendering to Lane**

Add to `Lane.tsx`, inside the messages `<div>`, after the message map — render artifacts and handoffs interleaved by timestamp:

```tsx
// Replace the messages-only rendering with a unified event list:

// Inside Lane component, replace the scrollable div content:

const laneEvents = events.filter((e) => {
  if (e.type === "message" && e.actor === agentId) return true;
  if (e.type === "artifact" && e.actor === agentId) return true;
  if (e.type === "handoff" && e.actor === agentId) return true;
  // Show incoming messages (where this agent is the recipient)
  if (e.type === "message" && (e.data as { to: string | null }).to === agentId) return true;
  return false;
});

// Then in the map, render differently based on type:
{laneEvents.map((event) => {
  if (event.type === "artifact") {
    const data = event.data as { filename: string };
    return (
      <div key={event.id} className={`text-xs p-2 rounded border ${colors.border} bg-gray-900/50`}>
        <span className="text-gray-500">artifact</span>{" "}
        <span className="text-gray-300">{data.filename}</span>
      </div>
    );
  }
  if (event.type === "handoff") {
    const data = event.data as { summary: string };
    return (
      <div key={event.id} className="text-xs p-2 rounded border border-gray-700 bg-gray-900/80">
        <div className="text-gray-500 mb-1">HANDOFF</div>
        <div className="text-gray-400 line-clamp-2">{data.summary}</div>
      </div>
    );
  }
  // message rendering (existing code)
  const data = event.data as { body: string; to: string | null; urgent: boolean };
  const isSent = event.actor === agentId;
  return (
    <div
      key={event.id}
      className={`text-xs p-2 rounded border ${
        isSent ? colors.border : "border-gray-700 border-dashed"
      } ${data.urgent ? "border-urgent bg-urgent/5" : isSent ? colors.bg : "bg-gray-900/30"} ${
        isSent ? "opacity-100" : "opacity-50"
      }`}
    >
      {isSent && data.to && (
        <div className="text-[10px] text-gray-500 mb-0.5">
          → {ROLE_LABELS[data.to] ?? data.to}
        </div>
      )}
      {!isSent && (
        <div className="text-[10px] text-gray-500 mb-0.5">
          ← {ROLE_LABELS[event.actor] ?? event.actor}
        </div>
      )}
      <div className="text-gray-300 line-clamp-3">{data.body}</div>
    </div>
  );
})}
```

**Step 2: Verify**

Expected: Artifact cards and handoff markers appear in the correct agent's lane alongside messages.

**Step 3: Commit**

```bash
git add -A
git commit -m "feat: artifact cards + handoff markers in lanes"
```

---

### Task 8: Message animations (framer-motion)

**Files:**
- Modify: `glink-replay/src/components/Lane.tsx` — wrap items in motion.div

**Step 1: Add entry animations**

Wrap each event item in the Lane with a `motion.div`:

```tsx
import { motion } from "framer-motion";

// Wrap each event rendered item:
<motion.div
  key={event.id}
  initial={{ opacity: 0, y: 8 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.2 }}
>
  {/* existing event content */}
</motion.div>
```

**Step 2: Auto-scroll to latest**

Add a ref to the scroll container and scroll to bottom when new events arrive:

```tsx
import { useRef, useEffect } from "react";

// Inside Lane component:
const scrollRef = useRef<HTMLDivElement>(null);

useEffect(() => {
  if (scrollRef.current) {
    scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
  }
}, [laneEvents.length]);

// On the scrollable div:
<div ref={scrollRef} className="flex-1 overflow-y-auto p-3 space-y-2">
```

**Step 3: Verify**

Expected: Messages animate in smoothly. Lanes auto-scroll to the latest message.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: message entry animations + auto-scroll"
```

---

### Task 9: Stats overlay

**Files:**
- Create: `glink-replay/src/components/StatsOverlay.tsx`
- Modify: `glink-replay/src/App.tsx`

**Step 1: Create StatsOverlay**

Create `glink-replay/src/components/StatsOverlay.tsx`:

```tsx
import { useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { useReplayStore } from "../engine/replay-store";
import type { AgentId } from "../types";

const AGENT_COLORS: Record<AgentId, string> = {
  "Architect-CEO": "bg-ceo",
  "Architect-CTO": "bg-cto",
  "Architect-COO": "bg-coo",
};

const ROLE_LABELS: Record<AgentId, string> = {
  "Architect-CEO": "CEO",
  "Architect-CTO": "CTO",
  "Architect-COO": "COO",
};

export function StatsOverlay() {
  const [open, setOpen] = useState(false);
  const visibleEvents = useReplayStore((s) => s.visibleEvents);
  const totalEvents = useReplayStore((s) => s.meta.totalEvents);

  const msgs = visibleEvents.filter((e) => e.type === "message");
  const decisions = visibleEvents.filter((e) => e.type === "decision");
  const artifacts = visibleEvents.filter((e) => e.type === "artifact");
  const handoffs = visibleEvents.filter((e) => e.type === "handoff");

  const agents: AgentId[] = ["Architect-CEO", "Architect-CTO", "Architect-COO"];
  const perAgent = agents.map((a) => ({
    id: a,
    count: msgs.filter((e) => e.actor === a).length,
  }));
  const maxCount = Math.max(...perAgent.map((a) => a.count), 1);

  return (
    <>
      <button
        onClick={() => setOpen(!open)}
        className="fixed top-3 right-3 px-3 py-1 text-xs bg-gray-800 rounded hover:bg-gray-700 text-gray-400 z-50"
      >
        Stats
      </button>
      <AnimatePresence>
        {open && (
          <motion.div
            initial={{ opacity: 0, x: 20 }}
            animate={{ opacity: 1, x: 0 }}
            exit={{ opacity: 0, x: 20 }}
            className="fixed top-12 right-3 w-56 bg-gray-900 border border-gray-800 rounded-lg p-4 z-50"
          >
            <div className="space-y-2 text-xs font-mono">
              <div className="flex justify-between text-gray-400">
                <span>Messages</span>
                <span className="text-white">{msgs.length}</span>
              </div>
              <div className="flex justify-between text-gray-400">
                <span>Decisions</span>
                <span className="text-decision">{decisions.length}</span>
              </div>
              <div className="flex justify-between text-gray-400">
                <span>Artifacts</span>
                <span className="text-white">{artifacts.length}</span>
              </div>
              <div className="flex justify-between text-gray-400">
                <span>Handoffs</span>
                <span className="text-white">{handoffs.length}</span>
              </div>
              <div className="flex justify-between text-gray-400">
                <span>Progress</span>
                <span className="text-white">
                  {visibleEvents.length}/{totalEvents}
                </span>
              </div>
              <div className="pt-2 border-t border-gray-800 space-y-1">
                {perAgent.map((a) => (
                  <div key={a.id} className="flex items-center gap-2">
                    <span className="text-gray-500 w-8">{ROLE_LABELS[a.id]}</span>
                    <div className="flex-1 h-2 bg-gray-800 rounded-full overflow-hidden">
                      <motion.div
                        className={`h-full ${AGENT_COLORS[a.id]} rounded-full`}
                        animate={{ width: `${(a.count / maxCount) * 100}%` }}
                        transition={{ duration: 0.3 }}
                      />
                    </div>
                    <span className="text-gray-500 w-6 text-right">{a.count}</span>
                  </div>
                ))}
              </div>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```

**Step 2: Add to App.tsx**

```tsx
import { StatsOverlay } from "./components/StatsOverlay";

// In the return, add before Timeline:
<StatsOverlay />
```

**Step 3: Verify**

Expected: "Stats" button in top-right. Clicking opens overlay with running counters and per-agent bar chart that animate as the replay progresses.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: stats overlay with running counters"
```

---

### Task 10: Event modal (click to expand)

**Files:**
- Create: `glink-replay/src/components/EventModal.tsx`
- Create: `glink-replay/src/engine/modal-store.ts`
- Modify: `glink-replay/src/components/Lane.tsx` — add click handler
- Modify: `glink-replay/src/App.tsx` — render modal

**Step 1: Create modal store**

Create `glink-replay/src/engine/modal-store.ts`:

```typescript
import { create } from "zustand";
import type { ReplayEvent } from "../types";

interface ModalState {
  event: ReplayEvent | null;
  open: (event: ReplayEvent) => void;
  close: () => void;
}

export const useModalStore = create<ModalState>((set) => ({
  event: null,
  open: (event) => set({ event }),
  close: () => set({ event: null }),
}));
```

**Step 2: Create EventModal**

Create `glink-replay/src/components/EventModal.tsx`:

```tsx
import { motion, AnimatePresence } from "framer-motion";
import { useModalStore } from "../engine/modal-store";
import type { MessageData, DecisionData, ArtifactData, HandoffData, CommentData } from "../types";

export function EventModal() {
  const { event, close } = useModalStore();

  return (
    <AnimatePresence>
      {event && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          onClick={close}
          className="fixed inset-0 bg-black/60 flex items-center justify-center z-50 p-8"
        >
          <motion.div
            initial={{ scale: 0.95 }}
            animate={{ scale: 1 }}
            exit={{ scale: 0.95 }}
            onClick={(e) => e.stopPropagation()}
            className="bg-gray-900 border border-gray-700 rounded-lg p-6 max-w-2xl w-full max-h-[80vh] overflow-y-auto"
          >
            <div className="flex items-center justify-between mb-4">
              <div>
                <span className="text-xs text-gray-500 uppercase">{event.type}</span>
                <span className="text-xs text-gray-600 ml-2">
                  {new Date(event.timestamp).toLocaleTimeString()}
                </span>
              </div>
              <button onClick={close} className="text-gray-500 hover:text-white">
                ✕
              </button>
            </div>
            <div className="text-xs text-gray-500 mb-2">
              From: {event.actor}
            </div>
            <div className="text-sm text-gray-200 whitespace-pre-wrap">
              {event.type === "message" && (event.data as MessageData).body}
              {event.type === "decision" && (event.data as DecisionData).title}
              {event.type === "artifact" && (event.data as ArtifactData).filename}
              {event.type === "handoff" && (event.data as HandoffData).summary}
              {event.type === "comment" && (event.data as CommentData).body}
            </div>
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

**Step 3: Add click handlers to Lane events**

In `Lane.tsx`, wrap each event item with `onClick={() => modalStore.open(event)}`:

```tsx
import { useModalStore } from "../engine/modal-store";

// Inside Lane:
const openModal = useModalStore((s) => s.open);

// On each event wrapper div, add:
onClick={() => openModal(event)}
className="... cursor-pointer hover:brightness-125"
```

**Step 4: Add EventModal to App.tsx**

```tsx
import { EventModal } from "./components/EventModal";

// In return, add at the end:
<EventModal />
```

**Step 5: Verify**

Expected: Clicking any message, artifact, or handoff opens a modal with the full content. Click outside or ✕ to close. Escape key closes.

**Step 6: Add Escape key handling**

In EventModal, add:
```tsx
import { useEffect } from "react";

// Inside EventModal:
useEffect(() => {
  function handleEsc(e: KeyboardEvent) {
    if (e.key === "Escape") close();
  }
  window.addEventListener("keydown", handleEsc);
  return () => window.removeEventListener("keydown", handleEsc);
}, [close]);
```

**Step 7: Commit**

```bash
git add -A
git commit -m "feat: event modal for full message view"
```

---

### Task 11: Polish + final integration

**Files:**
- Modify: `glink-replay/src/index.css` — scrollbar styling, final tweaks
- Modify: `glink-replay/index.html` — favicon, meta tags

**Step 1: Add custom scrollbar styles**

Add to `glink-replay/src/index.css`:

```css
/* Thin scrollbars */
.scrollbar-thin::-webkit-scrollbar {
  width: 4px;
}
.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent;
}
.scrollbar-thin::-webkit-scrollbar-thumb {
  background: #374151;
  border-radius: 2px;
}

/* Line clamp utilities */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

**Step 2: Update index.html meta**

```html
<title>GLink Replay — The Architects Founded a Company</title>
<meta name="description" content="Watch 3 AI agents found a company in 6 hours through GLink collaboration" />
<style>body { background: #030712; }</style>
```

**Step 3: Full build test**

```bash
cd /Users/shamim/Projects/glink-demo/the-architects/glink-replay
npm run build
npm run preview
```

Expected: Production build succeeds. Preview server shows the full working replay.

**Step 4: Final commit**

```bash
git add -A
git commit -m "feat: polish — scrollbars, meta, production build"
```

---

### Task 12: Verify the full experience

**Step 1: Fresh build + export**

```bash
cd /Users/shamim/Projects/glink-demo/the-architects/glink-replay
npm run export-data
npm run build
npm run preview
```

**Step 2: Test checklist**

- [ ] Press Space → replay starts
- [ ] Speed buttons (1x, 5x, 10x, 50x, 100x) work
- [ ] Scrubber is draggable
- [ ] Decision markers (gold dots) visible on scrubber
- [ ] Left/Right arrows skip between decisions
- [ ] Messages appear in correct lanes
- [ ] Lane headers pulse for active speaker
- [ ] Decision banners appear with gold accent
- [ ] Artifact cards appear in correct lanes
- [ ] Click any message → modal with full text
- [ ] Stats overlay shows running counters
- [ ] Per-agent bar chart animates
- [ ] Auto-scroll follows latest message
- [ ] 1-5 keys set speed presets
- [ ] Escape closes modal

**Step 3: Commit any fixes**

```bash
git add -A
git commit -m "chore: verification fixes"
```
