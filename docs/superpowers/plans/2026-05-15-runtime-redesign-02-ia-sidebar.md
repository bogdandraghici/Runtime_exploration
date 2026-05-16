# IA & Sidebar Nav — Plan 02

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the Runtime tab's information architecture and the left sidebar navigation that exposes it. Foundation for every page plan.

**Architecture:** 8-entry sidebar grouped semantically (Live state · Deployment · Work) with Overview at the top. Sidebar carries live status badges (counts, active build chip, pulse) so it doubles as a glanceable status surface. Footer hosts "Tour & help" anchor.

**Tech Stack:** Figma, FlowX DS tokens, Markdown spec doc.

**Deliverables:**
- Create: `docs/superpowers/specs/components/runtime-sidebar.md`
- Create: Figma frame "Runtime / Sidebar" with default + interaction states

---

### Task 1: Define information model

**Files:** create `docs/superpowers/specs/components/runtime-sidebar.md`

- [ ] **Step 1: Document the IA tree and badge inputs**

```markdown
## Information model

const NAV: NavGroup[] = [
  { kind: 'item', id: 'overview', label: 'Overview', icon: 'home', route: '/runtime/overview', badge: null },
  { kind: 'group', label: 'Live state', items: [
      { id: 'instances', label: 'Instances', icon: 'play', route: '/runtime/instances',
        badge: { kind: 'info', value: live.activeInstanceCount } },
      { id: 'incidents', label: 'Incidents', icon: 'alert', route: '/runtime/incidents',
        badge: { kind: 'alert', value: live.openIncidentCount } },
      { id: 'live-evals', label: 'Live Evals', icon: 'wave', route: '/runtime/live-evals',
        badge: { kind: 'warn', value: live.openAlertCount } },
  ]},
  { kind: 'group', label: 'Deployment', items: [
      { id: 'builds', label: 'Builds', icon: 'diamond', route: '/runtime/builds',
        badge: { kind: 'info', value: live.activeBuildTag } },
      { id: 'configuration', label: 'Configuration', icon: 'gear', route: '/runtime/configuration', badge: null },
      { id: 'triggers', label: 'Triggers', icon: 'loop', route: '/runtime/triggers', badge: null },
  ]},
  { kind: 'group', label: 'Work', items: [
      { id: 'tasks', label: 'Tasks', icon: 'checkbox', route: '/runtime/tasks',
        badge: { kind: 'info', value: live.pendingTaskCount } },
  ]},
];
```

Badge kinds: `'info' | 'warn' | 'alert' | 'success'`.

- [ ] **Step 2: Commit**

Commit message: `spec: define runtime sidebar information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document props, events, and live-data source**

```markdown
## Component API

Props: nav: NavGroup[]; activeRoute: string; live: LiveCounters; onNavigate: (route: string) => void;

LiveCounters source: subscribed websocket / SSE channel published by the same aggregator that feeds the Overview activity feed (see Plan 03). Counters refresh every 5s on the wire, debounced to 1s in the UI.

Behaviors:
- Active item highlight on prefix match (e.g. `/runtime/builds/v1.4.2` → Builds active)
- Hover state for non-active items
- Collapsible to icon-only via Cmd+\ (saved per-user)
- Footer: "All systems active" pulse (green dot) when openIncidentCount === 0 && openAlertCount === 0; otherwise "{n} incident(s) open" link
- Footer: persistent "Tour & help" entry (anchor for documentation layer — see Plan 11)
```

- [ ] **Step 2: Commit**

Commit message: `spec: define runtime sidebar component API`

---

### Task 3: Build hi-fi static mockup in Figma

- [ ] **Step 1: Create Figma frame**

Frame: "Runtime / Sidebar / Default". Width 224px. Show all 8 entries grouped with example badges populated. Footer pulse + tour link.

- [ ] **Step 2: Add collapsed variant**

Width 56px, icon-only, tooltips on hover.

- [ ] **Step 3: Paste Figma frame URLs into spec doc**

- [ ] **Step 4: Commit**

Commit message: `spec(sidebar): default + collapsed mockups`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Default | Initial load | As designed |
| Hover | Mouse over inactive item | Bg `color/surface/alt`, text `color/text/default` |
| Active | Route prefix matches | Bg `color/blue/50`, text `color/blue/700`, icon bg `color/blue/500` |
| Badge update | Live counter changes | Counter animates from old → new (200ms count-up) |
| Alert badge appears | openIncidentCount: 0 → N | Pulse animation 600ms once |
| Collapsed | User pressed Cmd+\ | Width 56px, labels hidden, tooltips on hover |
| Pulse (healthy) | openIncidentCount === 0 | Green dot, "All systems active" |
| Pulse (alert) | openIncidentCount > 0 | Red dot, "{n} incident(s) open" — link to Incidents |
| Mobile (<760px) | Viewport narrow | Sidebar hidden, hamburger in app-bar reveals drawer |
```

- [ ] **Step 2: Add Figma variants**

- [ ] **Step 3: Commit**

Commit message: `spec(sidebar): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document all labels and aria-labels**

```markdown
## Copy

- Group labels: "Live state", "Deployment", "Work"
- Item labels: "Overview", "Instances", "Incidents", "Live Evals", "Builds", "Configuration", "Triggers", "Tasks"
- Footer healthy: "All systems active"
- Footer alert (n=1): "1 incident open"
- Footer alert (n>1): "{n} incidents open"
- Tour link: "Tour & help"
- Collapse toggle aria-label: "Collapse sidebar" / "Expand sidebar"
- Badge tooltips: "Instances: 1,284 running", "Incidents: 4 open", "Live Evals: 2 alerts", "Builds: v1.4.2 active", "Tasks: 125 pending"
```

- [ ] **Step 2: Commit**

Commit message: `spec(sidebar): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: List tokens**

```markdown
## Tokens

- Sidebar bg: `color/surface/elevated`
- Right border: `color/border/default`
- Group label: `font/label/xs`, `color/text/dim`
- Item label: `font/body/sm`, `color/text/muted` (inactive), `color/blue/700` (active)
- Item active bg: `color/blue/50`
- Hover bg: `color/surface/alt`
- Badge info: `color/blue/50` / `color/blue/700`
- Badge warn: `color/orange/50` / `color/orange/700`
- Badge alert: `color/red/50` / `color/red/700`
- Pulse healthy: `color/green/500` with ring at 15% opacity
- Pulse alert: `color/red/500` with ring at 15% opacity
- Spacing: item 7px 12px padding, group label 12px 12px 6px, sidebar 14px 10px outer
```

- [ ] **Step 2: Commit**

Commit message: `spec(sidebar): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Write script**

5 tasks: (a) find where to view all running instances, (b) find what's broken, (c) identify the currently-active build without leaving the sidebar, (d) collapse and re-expand, (e) interpret the footer state.

- [ ] **Step 2: Run with 5 participants (2 ops, 2 dev, 1 support)**

- [ ] **Step 3: Capture findings into `## User test summary`**

- [ ] **Step 4: Commit**

Commit message: `spec(sidebar): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: Add `## Open questions for engineering`**

Initial items: live-counter websocket protocol, counter freshness SLA, badge animation library, accessibility focus order.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**

Commit message: `spec(sidebar): handoff to engineering`
