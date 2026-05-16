# Runtime Sidebar — Component Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

The Runtime Sidebar is one of the two foundation pieces of the Runtime tab redesign. It owns the information architecture for the entire tab and is the primary navigation surface that every page plan builds upon.

The sidebar exposes an 8-entry IA grouped semantically into three buckets — Live state, Deployment, and Work — with Overview pinned at the top. The grouping reflects how operators reason about the runtime: what is happening right now, what has been shipped, and what is queued.

Beyond navigation, the sidebar doubles as a glanceable status surface. Live badges (counts, the active build chip, and an alert pulse) keep operators oriented without forcing them onto a dashboard, and a persistent "Tour & help" footer anchor hosts the documentation layer.

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

Badge kinds: `'info' | 'warn' | 'alert' | 'success'`.

## Component API

Props: nav: NavGroup[]; activeRoute: string; live: LiveCounters; onNavigate: (route: string) => void;

LiveCounters source: subscribed websocket / SSE channel published by the same aggregator that feeds the Overview activity feed (see Plan 03). Counters refresh every 5s on the wire, debounced to 1s in the UI.

Behaviors:
- Active item highlight on prefix match (e.g. `/runtime/builds/v1.4.2` → Builds active)
- Hover state for non-active items
- Collapsible to icon-only via Cmd+\ (saved per-user)
- Footer: "All systems active" pulse (green dot) when openIncidentCount === 0 && openAlertCount === 0; otherwise "{n} incident(s) open" link
- Footer: persistent "Tour & help" entry (anchor for documentation layer — see Plan 11)

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

## Copy

- Group labels: "Live state", "Deployment", "Work"
- Item labels: "Overview", "Instances", "Incidents", "Live Evals", "Builds", "Configuration", "Triggers", "Tasks"
- Footer healthy: "All systems active"
- Footer alert (n=1): "1 incident open"
- Footer alert (n>1): "{n} incidents open"
- Tour link: "Tour & help"
- Collapse toggle aria-label: "Collapse sidebar" / "Expand sidebar"
- Badge tooltips: "Instances: 1,284 running", "Incidents: 4 open", "Live Evals: 2 alerts", "Builds: v1.4.2 active", "Tasks: 125 pending"

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

## Open questions for engineering

- Live-counter websocket protocol
- Counter freshness SLA
- Badge animation library
- Accessibility focus order

## Figma + user testing — pending

- TODO: Build Figma frames (default expanded, collapsed, hover, active, badge update, alert pulse, mobile drawer).
- TODO: Run user test with 5 participants per Plan 02 Task 7 script.
