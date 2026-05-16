# Incidents — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

Incidents is the unified "what's broken" surface for Runtime. It collapses four formerly separate streams — Failed Process Start, Failed Triggers, Exceptions on running Process Instances, and AI scorer breaches from Live Evals — into a single page. This resolves friction #5 from the design spec: today users hunt across three surfaces (plus Live Evals) to triage failures; tomorrow they land here.

The page is organized around a **four-stage taxonomy**: Pre-start, Trigger, Mid-execution, and AI quality. Stage tabs at the top act as the primary filter. Rows carry a severity band, a status pill, and a build chip so the list reads as a glanceable triage queue. A shared detail panel exposes root cause, stack trace, a merged timeline that interleaves exception events with build deploys for correlation, and related entities for cross-navigation.

AI quality breaches appear here because, from an ops perspective, a breached scorer IS an incident — but Live Evals remains the canonical config and time-series home for scorers, and rows in this stage deep-link out. The workflow stays lightweight (Open · Ack · Resolved); this is a triage surface, not a ticketing system.

## Information model

interface IncidentRow {
  id: string;
  stage: 'pre-start' | 'trigger' | 'mid-execution' | 'ai-quality';
  severity: 'critical' | 'high' | 'medium' | 'low';
  status: 'open' | 'ack' | 'resolved';
  title: string;
  subline: string;
  sourceEntity: { kind: 'instance'|'trigger'|'workflow'|'scorer'; id: string };
  buildTag: string;
  firstRaisedAt: string;
  recurrenceCount: number;
  assignedTo?: string;
}

interface IncidentDetail extends IncidentRow {
  rootCause: { exceptionType?: string; location?: string; message?: string; node?: string; tokenId?: string; firstRaisedAt; recurrenceCount };
  stackTrace?: string;
  timeline: TimelineEvent[];           // merged: exception + token movements + build deploys
  relatedEntities: RelatedEntity[];    // upward/downward/sideways links
  similarIncidents: IncidentRow[];
}

## Component API

List props: rows; filters; total; onRowClick; onFilterChange; onStageChange; loading.

Stage tab acts as primary filter; secondary filters: severity, status, build, source, time window.

URL: `/runtime/incidents?stage=&sev=&status=&build=&src=&t=24h`. Detail open: `…?detail={id}`. Maximized: `/runtime/incidents/{id}`.

Actions on detail: Acknowledge (status: open→ack, sets assignedTo to current user if unset), Assign to me, Resolve (status: ack→resolved). Auto-resolve: backend may flip ack→resolved when the underlying condition clears for N minutes (configurable, default 10).

AI quality stage rows: sourceEntity.kind === 'scorer'; clicking deep-links to Live Evals scorer detail. The Live Evals breach record (LiveAlertState) is the canonical source.

## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Open badge pulse | New incident appears | Stage tab counter animates, sidebar badge pulses |
| Severity scan | Default scroll | 4px left bar gives at-a-glance severity |
| Action — Ack | Click Acknowledge | Status pill animates open→ack, button stack reduces |
| Action — Assign | Click Assign to me | Adds assignee chip, no status change |
| Action — Resolve | Click Resolve | Status pill ack→resolved (green), row de-emphasizes |
| Stage filter | Tab click | List rerenders, URL updates |
| Empty stage | Stage has no incidents | "No {stage} incidents in {window} — nice." (positive empty state) |
| Empty all | No incidents | "All clear — no open incidents." (full positive state, green border) |

## Copy

- Page title: "Incidents"
- Help icon: "Anything that broke or breached — across the full lifecycle."
- Stage tab labels: "All", "Pre-start", "Trigger", "Mid-execution", "AI quality"
- Stage tooltips: pre-start "Process refused to start — validation, missing input, etc.", trigger "Trigger fired but didn't reach a process start", mid-execution "Process instance threw mid-run", ai-quality "An AI scorer crossed its threshold"
- Severity labels in detail: "Critical", "High", "Medium", "Low"
- Status labels: "Open", "Ack", "Resolved"
- Actions: "Acknowledge", "Assign to me", "Resolve"
- Detail tabs: "Context", "Stack trace", "Timeline", "Similar"
- Doc banner (mid-exec): "About this incident. A 'Mid-execution' incident means a process instance threw while running. The process is paused at the offending node; you can retry, skip, or terminate from the parent instance page."
- Empty all: "All clear — no open incidents."
- Empty stage: "No {stage label} incidents in {window} — nice."
- Resolve confirmation: "Mark this incident as resolved? The underlying condition isn't auto-checked — you're asserting it's handled."

## Tokens

- Severity bars: critical `color/red/500`, high `color/orange/500`, medium `color/yellow/500`, low `color/text/dim`
- Stage pills: pre-start `color/red/50`/`color/red/700`, trigger `color/yellow/50`/`color/yellow/700`, mid-exec `color/orange/50`/`color/orange/700`, ai-quality `color/purple/50`/`color/purple/500`
- Status pills: open red, ack yellow, resolved green (matches Instances 5-state palette but maps onto incident statuses)
- Timeline dot: default `color/text/dim`, bad-event `color/red/500`
- Stack trace block: `#1d232c` bg, monospace, `color/orange/500` for exception lines

## Open questions for engineering

- Severity assignment logic (auto vs manual)
- Auto-resolve threshold
- Similar-incidents matching algorithm
- LiveAlertState → Incident projection (read-side join or backend push?)
- Retry/skip/terminate actions for mid-execution (likely link out to instance detail)

## Figma + user testing — pending

- [ ] Build hi-fi static mockup — "Incidents / List" frame with all four stages represented (1 mid-execution selected · 2 ai-quality · 2 trigger · 1 pre-start · 1 resolved); stage tabs visible with counts (Mid-execution alert, AI quality warn)
- [ ] Build hi-fi static mockup — "Incidents / Detail" frame showing Root cause panel · Related entities (3 cards: failed instance, parent session, build) · Timeline (merged events including a deploy marker 11m ago)
- [ ] Add Figma variants for each interaction state in the matrix above
- [ ] User test script: 5 tasks — (a) tell me what's most critical, (b) walk me through how this happened (use Timeline + Related entities), (c) acknowledge an incident and assign yourself, (d) filter to only AI quality, (e) find an incident's parent session
- [ ] Run user test with 5 participants
- [ ] Capture findings under a `## User test summary` section and iterate
