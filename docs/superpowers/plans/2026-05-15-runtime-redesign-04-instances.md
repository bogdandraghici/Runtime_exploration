# Instances Page — Plan 04

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the merged Instances page — Process Instances + UI Flow Sessions on one list with one shared detail panel. Resolves friction #2, #4 (partial), #7 (status), #9 (Workflows tab redundancy).

**Architecture:** Single list with type column, filters, normalized status pill. Right-slide detail panel (consumes the shared detail-panel component from Plan 01) with adaptive tabs: Overview · Variables · Activity tree (type-adaptive content) · AI runs (embedded Live Evals slice) · Incidents · Audit.

**Tech Stack:** Figma, FlowX DS tokens, Markdown spec doc.

**Deliverables:**
- Create: `docs/superpowers/specs/pages/instances.md`
- Create: Figma frames "Runtime / Instances / List", "Runtime / Instances / Detail panel — Process variant", "… UI Flow variant"

**Depends on:** Plan 01 (detail panel), Plan 02 (sidebar).

---

### Task 1: Define information model

- [ ] **Step 1: Document the data contracts including status normalization**

```markdown
## Information model

interface InstanceRow {
  type: 'process' | 'uiflow';
  id: string;
  definitionName: string;
  status: NormalizedStatus;
  rawStatus: string;             // original ProcessState or session.status
  buildTag: string;
  buildId: string;
  atStep: string;                // current node label / step descriptor
  startedAt: string;
  spawnedBy?: { type: 'uiflow'; id: string };
  spawnedCount?: number;         // for uiflow rows
  activeIncidentCount?: number;
}

type NormalizedStatus = 'running' | 'waiting' | 'completed' | 'failed' | 'terminated';

const STATUS_MAP: Record<ProcessState | UiFlowStatus, NormalizedStatus> = {
  STARTED: 'running', ON_HOLD: 'waiting',
  FINISHED: 'completed',
  FINISHED_WITH_ERROR: 'failed', FAILED: 'failed',
  ABORTED: 'terminated', EXPIRED: 'terminated', TERMINATED: 'terminated', DISMISSED: 'terminated',
  CREATED: 'running',
  // UI flow session statuses map similarly
};

interface InstanceDetail {
  row: InstanceRow;
  overview: { startedAt; updatedAt; definitionName; buildAtStart; spawnedBy?; spawnedProcesses?: InstanceRow[] };
  variables: Record<string, unknown>;
  activityTree: ActivityNode[];   // type-adaptive content
  aiRuns: LiveEvalsSlice;          // embedded from Live Evals
  incidents: IncidentSummary[];
  audit: AuditEvent[];
}
```

- [ ] **Step 2: Commit**: `spec: instances information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document list + detail props and URL contract**

```markdown
## Component API

List props: rows: InstanceRow[]; filters: Filters; total: number; onRowClick: (row) => void; onFilterChange; onLoadMore; loading: boolean;

Filters: { type: 'all'|'process'|'uiflow'; status: NormalizedStatus[]; build: string[]; definition: string[]; variableKv?: {k,v}; search?: string; timeWindow }.

URL contract:
- List view: `/runtime/instances?type=&status=&build=&def=&kv=&q=&t=24h`
- Detail open: `…?detail={id}&tab={tabId}`
- Detail maximized: `/runtime/instances/{id}/{tabId}`

Tab visibility rules:
- "Activity tree" tab: always present; content adapts to row.type
- "Incidents" tab: only when row.activeIncidentCount > 0
- "AI runs" tab: always present (empty state when no runs)

Cross-link: a Process row with row.spawnedBy renders inline "↑ spawned by sess_xxxx" with click-through to the parent session.
```

- [ ] **Step 2: Commit**: `spec: instances component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: List view mockup**

Figma frame "Runtime / Instances / List": 6 rows mixed types (2 UI Flow including selected, 3 Process including one anchor-row "spawned by"), filters bar, type column with green P / purple F badges, normalized status pills (5 variants visible), build chips.

- [ ] **Step 2: Detail — Process variant**

Frame: process instance with Activity tree showing tokens + subprocesses, Incidents tab present (badge 3), AI runs tab populated.

- [ ] **Step 3: Detail — UI Flow variant**

Frame: UI flow session with Activity tree showing spawned process(es), no Incidents tab.

- [ ] **Step 4: Commit**: `spec(instances): list + two detail variants`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Row hover | Mouse over | Bg `color/surface/alt` |
| Row selected | Detail open | Bg `color/blue/50` |
| Anchor row | row has spawnedBy match in current page | Bg `color/yellow/50`, arrow indicator |
| Status pill hover | Hover any pill | Tooltip shows raw status (e.g. "FINISHED_WITH_ERROR · raised exception during execution") |
| Live update | Status changes for a row in view | Pill animates color shift over 400ms |
| Filter changed | onFilterChange | Table reshuffles with 200ms transition; URL updates |
| Empty filter result | rows.length === 0 with filters active | "No instances match these filters — clear filters or expand time window" |
| Empty no filters | rows.length === 0 with no filters | "No active instances yet — when a UI flow runs or a process starts, it'll appear here" |
| Detail open | onRowClick | Right-slide panel from Plan 01 |
| Mobile (<760px) | Viewport narrow | List collapses to cards; detail becomes full-screen modal |
```

- [ ] **Step 2: Add Figma variants for status pill hover, anchor row, and empty states**

- [ ] **Step 3: Commit**: `spec(instances): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document copy**

```markdown
## Copy

- Page title: "Instances"
- Help icon: "Any user-facing session or engine-driven process running in this app."
- Filter labels: "Type", "Status", "Build", "Definition", "Variable", with search placeholder "Search by ID, variable…"
- Column headers: "Type", "Identity", "Status", "Build", "At step", "Started"
- Status pill labels: "Running", "Waiting", "Completed", "Failed", "Terminated"
- Status raw tooltips (examples): "STARTED · executing", "ON_HOLD · paused awaiting input", "FINISHED_WITH_ERROR · raised exception during execution"
- Spawn anchor: "↑ spawned by {parentId}"
- Empty (no filters): "No active instances yet — when a UI flow runs or a process starts, it'll appear here."
- Empty (filtered): "No instances match these filters."
- Detail crumb: "Instance details"
- Tab labels: "Overview", "Variables", "Activity tree", "AI runs", "Incidents", "Audit"
- Activity tree doc banner: "What's an Activity tree? For UI Flow sessions, this is the list of process instances the session spawned. For processes, it shows active tokens and subprocesses. Same idea, different content."
- AI runs deep-link CTA: "Open in Live Evals →"
```

- [ ] **Step 2: Commit**: `spec(instances): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: List tokens**

```markdown
## Tokens

- Type badge process: `color/green/500` bg, white text
- Type badge uiflow: `color/purple/500` bg, white text
- Anchor row: `color/yellow/50` bg
- Build chip: `color/blue/50` bg, `color/blue/700` text, `color/blue/100` border
- Status pill — running: `color/green/50` / `color/green/700` with dot `color/green/500` + pulse
- Status pill — waiting: `color/yellow/50` / `color/yellow/700`
- Status pill — completed: `color/surface/alt` / `color/text/muted`
- Status pill — failed: `color/red/50` / `color/red/700`
- Status pill — terminated: `color/surface/alt` / `color/text/muted` darker
- Table row padding: 11px 14px
```

- [ ] **Step 2: Commit**: `spec(instances): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Script**

6 tasks: (a) find a UI flow session for user X, (b) tell me if it failed and why, (c) navigate from a session to a process it spawned, (d) explain what "Waiting" means here, (e) filter to only Failed instances on v1.4.2, (f) view an instance's AI run quality scores.

- [ ] **Step 2: Run with 5 participants**

- [ ] **Step 3: `## User test summary`**

- [ ] **Step 4: Commit**: `spec(instances): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: Add `## Open questions for engineering`**

Items: status normalization location (UI mapper vs backend new field), spawn-link query performance (does API need a new endpoint?), AI runs slice contract (Live Evals API extension?), variable-search backend index, default time window.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(instances): handoff`
