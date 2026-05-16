# Instances — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

The Instances page merges what used to be two separate surfaces — Process Instances and UI Flow Sessions — into a single unified list backed by one shared detail panel. A type column distinguishes process rows from UI flow rows, filters narrow by type, status, build, definition, and variables, and a normalized status pill replaces the divergent raw status vocabularies of the two underlying systems.

Selecting any row opens the shared right-slide detail panel (defined in Plan 01) with adaptive tabs: Overview, Variables, Activity tree (content adapts to row type), AI runs (embedded Live Evals slice), Incidents (conditional), and Audit. Process and UI flow detail views share chrome but diverge in the Activity tree content and in whether the Incidents tab appears.

This page resolves friction #2 (split between Process Instances and UI Flow Sessions), friction #4 (partial — observability of spawn relationships), friction #7 (status vocabulary normalization across the two systems), and friction #9 (Workflows tab redundancy collapsed into this single list).

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

## Open questions for engineering

- Status normalization location (UI mapper vs backend new field)
- Spawn-link query performance (does API need a new endpoint?)
- AI runs slice contract (Live Evals API extension?)
- Variable-search backend index
- Default time window

## Figma + user testing — pending

- TODO: Build Figma frames (List view, Detail Process variant, Detail UI Flow variant, anchor row, empty states, status pill hover, mobile).
- TODO: Run user test with 5 participants per Plan 04 Task 7 script.
