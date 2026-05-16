# Overview Dashboard — Plan 03

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the new activity-first Overview landing — the dashboard that gives "real insight into runtime in five seconds."

**Architecture:** Conditional incident banner at top → dominant activity feed (left, ~65% width) → stacked KPI cards (right column) → trends strip at the bottom. Strong visual hierarchy: feed is protagonist, KPI cards subordinate, trends peripheral.

**Tech Stack:** Figma, FlowX DS tokens, Markdown spec doc.

**Deliverables:**
- Create: `docs/superpowers/specs/pages/overview.md`
- Create: Figma frames "Runtime / Overview" (healthy, alert, loading, empty)

**Depends on:** Plan 02 (sidebar). Optionally Plan 01 (detail panel for deep-link from feed rows).

---

### Task 1: Define information model

- [ ] **Step 1: Document the page's data contracts**

```markdown
## Information model

interface OverviewData {
  banner: IncidentBanner | null;     // null when healthy
  feed: ActivityEvent[];             // paginated, infinite scroll
  feedFilters: { types: EventType[]; search?: string; timeWindow: '1h'|'24h'|'custom' };
  inFlight: { processes: number; sessions: number; tasks: number; trend: TrendDirection };
  deployment: { activeBuildTag: string; activeBuildId: string; coverage: BuildCoverage[]; deployedAgo: string };
  aiQuality: ScorerMovement[];       // top 4 scorer movements vs previous build
  trends: { throughputPerMin: SparkSeries; p95Latency: SparkSeries; errorsPerHour: SparkSeries; costPerHour: SparkSeries };
}

type EventType = 'start' | 'finish' | 'incident' | 'deploy' | 'trigger' | 'ai-alert';
interface ActivityEvent { id; ts; type; severity?; title; subline; sourceEntity: { kind; id }; buildTag?; }
```

- [ ] **Step 2: Commit**: `spec: overview information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document props, events, and source streams**

```markdown
## Component API

Props: data: OverviewData; onEventClick: (e: ActivityEvent) => void; onKpiDrillIn: (kpi: 'inFlight'|'deployment'|'ai') => void; onTrendClick: (t: TrendKey) => void; onBannerReview: () => void;

Live stream: SSE/websocket channel "runtime.activity" — same aggregator that feeds the sidebar counters. Filter chips affect client-side rendering only; the stream is unfiltered.

URL contract: time window is `?t=24h`; filters are `?types=incident,deploy`; clicking a feed row pushes the entity route with the panel open.

Empty state: when feed has no events in selected window, show "No activity in the last {window} — adjust the time window or check that triggers are firing."
```

- [ ] **Step 2: Commit**: `spec: overview component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: Create Figma frame "Overview / Healthy"**

No banner, populated feed (7+ rows including day-divider rhythm), three KPI cards, trends strip with sparklines. 1440px breakpoint primary; 1280px responsive variant.

- [ ] **Step 2: Create Figma frame "Overview / Alert"**

Banner visible with 2-scorer-breached + 4-incident summary and "Review →" CTA. Feed shows the related events at the top.

- [ ] **Step 3: Paste Figma URLs into spec under `## Mockups`**

- [ ] **Step 4: Commit**: `spec(overview): healthy + alert mockups`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Healthy | banner === null | No banner; layout starts at feed |
| Alert | banner !== null | Banner visible, slides in from top on first appearance |
| Filter applied | User clicks a chip | Chip turns blue, feed rerenders, URL updates |
| Live append | New event arrives | Insert at top of feed with 300ms fade-in; if user scrolled below "Now" divider, show "↑ N new events" pill |
| Loading | initial mount | Skeleton: banner placeholder hidden, 6 feed skeleton rows, KPI cards shimmer |
| Empty feed | feed.length === 0 | Empty-state explanation (see Plan 11) |
| Drill-in | onEventClick | Pushes route, opens detail panel |
| Mobile (<760px) | Viewport narrow | KPI cards stack above feed; trends strip 2x2 grid |
```

- [ ] **Step 2: Add Figma variants for each non-trivial state**

- [ ] **Step 3: Commit**: `spec(overview): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document copy strings**

```markdown
## Copy

- Page title: "Overview"
- Help icon tooltip: "Real-time view of what's happening across this app right now."
- Filter chip labels: "All", "Starts", "Finishes", "Incidents", "Deploys", "Triggers", "AI alerts"
- Day dividers: "Now", "Earlier today", "Yesterday", "Earlier this week", "{date}"
- Banner title pattern: "{n} scorers breached · {m} active incidents · {k} failed starts in last hour"
- Banner CTA: "Review →"
- KPI titles: "In flight", "Deployment", "AI quality"
- "↑ N new events" live-append pill copy
- Empty feed: "No activity in the last {window} — adjust the time window or check that triggers are firing."
- Search placeholder: "Search by ID, variable, message…"
```

- [ ] **Step 2: Commit**: `spec(overview): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: List tokens**

```markdown
## Tokens

- Banner gradient: `linear-gradient(90deg, color/red/50 0%, color/orange/50 100%)`
- Banner left border: `color/red/500`, 4px
- Feed row hover: `color/surface/alt`
- Glyph backgrounds: start `color/green/500`, finish `color/text/muted`, error `color/red/500`, alert `color/orange/500`, deploy `color/purple/500`, trigger `color/yellow/500`
- Day-divider: `color/surface/alt` bg, `color/text/dim` text
- KPI big number: `font/heading/lg`
- Spark colors: throughput `color/blue/500`, p95 `color/orange/500`, errors `color/red/500`, cost `color/green/500`
- Spacing: card padding 12px 14px, feed row 9px 14px, banner 12px 16px
```

- [ ] **Step 2: Commit**: `spec(overview): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Write script**

6 tasks: (a) describe what's broken (banner test), (b) find a specific event from 15 min ago (feed scrolling/search), (c) tell me the currently active build, (d) tell me which scorer regressed most, (e) filter to only see deploys, (f) drill in to a feed row and come back.

- [ ] **Step 2: Run with 5 participants (2 ops, 2 dev, 1 support)**

- [ ] **Step 3: Add `## User test summary`**

- [ ] **Step 4: Commit**: `spec(overview): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: Add `## Open questions for engineering`**

Initial items: feed retention window default, SSE reconnection strategy, banner derivation logic (who decides "alert"?), KPI trend baseline (24h avg vs prev period).

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(overview): handoff to engineering`
