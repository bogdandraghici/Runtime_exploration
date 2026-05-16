# Triggers — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

Triggers combines what were three separate surfaces — Scheduled Processes, Manage Triggers, and their failure history — under one sidebar entry. The consolidation reflects how operators actually think about this domain: a trigger is a trigger regardless of whether it fires on a cron, a webhook, or an event, and its failure history is part of its identity rather than a separate concern.

The page is organized into four tabs: Scheduled, External, Recent fires, and Failed fires. The first two are configuration views (the old Scheduled Processes and Manage Triggers, respectively). The last two are execution-history views — Recent fires shows the cross-trigger fire log, and Failed fires narrows that log to fires that never reached a successful process start.

Trigger-stage failures double-surface in the Incidents page so that on-call operators can see all four incident stages in one list. This page provides the trigger-centric view: configuration alongside fire history, with deep-links into Incidents when a failed fire has been escalated.

## Information model

interface Trigger {
  id: string;
  name: string;
  type: 'cron' | 'webhook' | 'event';
  schedule?: string;             // cron expression or webhook description
  spawnsDefinitionName: string;
  spawnsDefinitionId: string;
  lastFireAt?: string;
  status: 'active' | 'paused' | 'erroring';
  errorCount24h: number;
  audit: AuditFields;
}

interface TriggerFire {
  id: string;
  triggerId: string;
  triggerName: string;
  triggerType: Trigger['type'];
  firedAt: string;
  outcome: 'success' | 'failed';
  spawnedInstanceId?: string;     // only when outcome === 'success'
  failureReason?: string;         // only when outcome === 'failed'
}

interface TriggerDetail extends Trigger {
  recentFires: TriggerFire[];
  failuresLast7d: TriggerFire[];
}

## Component API

Page props: tabs ('scheduled'|'external'|'recent-fires'|'failed-fires'), rows, filters, onRowClick, onPauseToggle, onTabChange.

URL: `/runtime/triggers?tab=scheduled&q=&status=`. Detail: `…?detail={triggerId}` (right-slide panel) or `/runtime/triggers/{triggerId}` (maximized).

Tab content:
- Scheduled tab: Trigger rows where type === 'cron', columns: Name · Type · Schedule · Spawns · Last fire · Status (active/paused with toggle)
- External tab: Trigger rows where type !== 'cron', columns same but Schedule → "on event"/"webhook"
- Recent fires tab: TriggerFire rows across all triggers, columns: Trigger name · Type · Fired at · Outcome · Spawned instance link
- Failed fires tab: TriggerFire rows where outcome === 'failed', columns same as Recent + Failure reason

A failed fire in this list double-surfaces in Incidents page (stage: trigger). Clicking a failed-fire here can deep-link to the Incidents detail for the same record if it has been escalated.

## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Active row | status === 'active' | Green pill "Active" |
| Paused row | status === 'paused' | Grey pill "Paused" |
| Erroring row | errorCount24h > 0 | Row bg `color/red/50`, red pill with count |
| Pause toggle | Click switch | Optimistic update, error toast on failure |
| Tab switch | Click tab | URL updates, content rerenders |
| Empty Scheduled | No cron triggers | "No scheduled processes yet — schedule a process to run on a cadence." |
| Empty External | No external triggers | "No external triggers configured — set up a webhook or event subscription." |
| Empty Failed fires | No failures in window | "No trigger failures in {window} — triggers are healthy." (positive empty) |

## Copy

- Page title: "Triggers"
- Help icon: "Scheduled processes + external triggers + their failure history, one page."
- Tab labels: "Scheduled", "External", "Recent fires", "Failed fires"
- Column headers: "Name", "Type", "Schedule", "Spawns", "Last fire", "Status"
- Type labels: "Cron", "Webhook", "Event"
- Status pills: "Active", "Paused", "{n} failure(s) (active)"
- Outcome labels: "Success", "Failed"
- Doc banner (cross-page note): "What's combined here. Old Scheduled Processes + old Manage Triggers + the trigger-failure subset of Failed Triggers. Mid-execution / Pre-start failures live in Incidents. The 'Failed fires' tab here is just for fires that didn't reach a process start."
- Empty Scheduled: "No scheduled processes yet — schedule a process to run on a cadence."
- Empty External: "No external triggers configured — set up a webhook or event subscription."
- Empty Failed fires: "No trigger failures in {window} — triggers are healthy."
- Pause confirmation: "Pause this trigger? It won't fire until you resume."
- Detail tabs: "Configuration", "Recent fires", "Failures"

## Tokens

- Active status pill: `color/green/50` / `color/green/700`
- Paused status pill: `color/surface/alt` / `color/text/muted`
- Erroring row bg: `color/red/50`
- Erroring status pill: `color/red/50` / `color/red/700`
- Cron schedule cell: monospace (`font/code/xs`)
- Outcome success: `color/green/700`
- Outcome failed: `color/red/700`

## Open questions for engineering

- Pause/resume API contract.
- Webhook signature verification UX.
- Replay-failed-fire affordance (future).
- How Failed fires here are linked to Incidents records.

## Figma + user testing — pending

- [ ] Figma "Triggers / Scheduled" frame with 3–4 cron rows including one paused and one with active failures (red-bg row).
- [ ] Figma "Triggers / External" frame with webhooks and events, including the Salesforce sync row with active failures.
- [ ] Figma "Triggers / Recent fires" and "Triggers / Failed fires" frames showing time-ordered fire history with outcome columns.
- [ ] Figma "Triggers / Detail (any tab)" frame.
- [ ] Run 5-task user test (find a scheduled process and pause it; explain why a trigger is erroring; view recent fires for "Salesforce sync"; navigate from a failed fire to its Incident record; identify which trigger spawned a given instance) with 5 participants emphasizing ops + admin personas.
- [ ] Capture `## User test summary` and iterate.
