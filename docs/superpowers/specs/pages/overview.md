# Overview Dashboard — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

The Overview Dashboard is the new activity-first landing page for Runtime, designed to deliver real insight into runtime in five seconds. Where the previous landing experience scattered attention across static KPI tiles, this redesign elevates a live activity feed as the page's protagonist and subordinates everything else around it.

The architecture is deliberately hierarchical: a conditional incident banner sits at the top, only appearing when something is wrong; the dominant activity feed occupies roughly 65% of the width on the left; stacked KPI cards (In flight, Deployment, AI quality) anchor the right column; and a trends strip with sparklines sits at the bottom as peripheral context. The feed is the protagonist, KPI cards are subordinate, trends are peripheral.

This page depends on the redesigned sidebar (Plan 02) and optionally on the detail panel (Plan 01) for deep-link behavior when users drill into a feed row.

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

## Component API

Props: data: OverviewData; onEventClick: (e: ActivityEvent) => void; onKpiDrillIn: (kpi: 'inFlight'|'deployment'|'ai') => void; onTrendClick: (t: TrendKey) => void; onBannerReview: () => void;

Live stream: SSE/websocket channel "runtime.activity" — same aggregator that feeds the sidebar counters. Filter chips affect client-side rendering only; the stream is unfiltered.

URL contract: time window is `?t=24h`; filters are `?types=incident,deploy`; clicking a feed row pushes the entity route with the panel open.

Empty state: when feed has no events in selected window, show "No activity in the last {window} — adjust the time window or check that triggers are firing."

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

## Tokens

- Banner gradient: `linear-gradient(90deg, color/red/50 0%, color/orange/50 100%)`
- Banner left border: `color/red/500`, 4px
- Feed row hover: `color/surface/alt`
- Glyph backgrounds: start `color/green/500`, finish `color/text/muted`, error `color/red/500`, alert `color/orange/500`, deploy `color/purple/500`, trigger `color/yellow/500`
- Day-divider: `color/surface/alt` bg, `color/text/dim` text
- KPI big number: `font/heading/lg`
- Spark colors: throughput `color/blue/500`, p95 `color/orange/500`, errors `color/red/500`, cost `color/green/500`
- Spacing: card padding 12px 14px, feed row 9px 14px, banner 12px 16px

## Open questions for engineering

- Feed retention window default
- SSE reconnection strategy
- Banner derivation logic (who decides "alert"?)
- KPI trend baseline (24h avg vs prev period)

## Figma + user testing — pending

- [ ] TODO: Build hi-fi static mockups — Figma frames "Overview / Healthy" (no banner, populated feed with 7+ rows, three KPI cards, trends strip; 1440px primary, 1280px responsive variant) and "Overview / Alert" (banner visible with 2-scorer-breached + 4-incident summary and "Review →" CTA; feed shows related events at top). Paste Figma URLs into spec under `## Mockups`.
- [ ] TODO: Run user test — 6 tasks with 5 participants (2 ops, 2 dev, 1 support): (a) describe what's broken (banner test), (b) find a specific event from 15 min ago (feed scrolling/search), (c) tell me the currently active build, (d) tell me which scorer regressed most, (e) filter to only see deploys, (f) drill in to a feed row and come back. Add findings under `## User test summary`.
