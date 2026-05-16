# Builds — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

Builds is the artifact list for every published app snapshot, but in the redesign it gains a first-class deployment lens that answers "who is running this build right now?" without forcing a detour through Configuration. Each row carries a coverage bar with traffic percentage alongside the live stats (runs in the last hour, errors with delta, average AI quality with delta, open incident count), so the question of which artifact matters most is legible from the list itself. This resolves friction #6 from the redesign spec.

The detail panel extends the same lens into depth. A new Deployment tab decomposes coverage into the default policy plus each override rule's slice, and explicitly names the roles or users that are *not* on this build. Comparison, Live runs, Workflows, and Changelog tabs round out the picture so the build can be evaluated as an artifact, as a deployed surface, and as a delta from its predecessor in one place.

Branch tabs (All, main, staging, beta) act as the primary filter. The page coordinates with Live Evals for comparison data and with Configuration for coverage resolution; both relationships are read-only from the Builds page.

## Information model

interface BuildRow {
  tag: string;
  id: string;
  branch: string;
  cutAt: string;
  cutBy: string;
  isActive: boolean;            // currently used by some policy
  isArchived: boolean;
  coverage: { percent: number; userCount: number; totalUsers: number };
  liveStats: { runsLastHour: number; errors: number; errorsDelta?: number; avgQuality?: number; qualityDelta?: number; openIncidentCount: number };
}

interface BuildDetail extends BuildRow {
  servedBy: PolicyRule[];        // who's serving this build
  notServedBy: { role: string; alternativeBuild: string }[]; // who isn't
  comparison?: BuildComparison;  // vs previous build on this branch
  liveRuns: InstanceRow[];       // active instances on this build
  workflows: WorkflowSummary[];
  changelog: ChangelogEntry[];
}

## Component API

List props: rows; branchTabs; filters; onRowClick; onBranchChange; onCompareClick(leftBuildId, rightBuildId).

URL: `/runtime/builds?branch=&status=&date=&author=`. Detail: `…?detail={buildId}&tab={tabId}`. Comparison: `/runtime/builds/{leftId}/compare/{rightId}` (re-uses existing Live Evals comparison route shape).

Actions on detail: "Compare to {selectable build}", "View in Live Evals", "Rollback" (gated by permission).

Coverage source: read-side join from Active Policy resolutions sampled over last 1h. Approximate by default; the spec doc must note that exact computation can be expensive.

Empty coverage: when percent === 0, render greyed bar with "Not currently serving traffic. Last served: {ago | never}."

## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Active build row | row.isActive | Green "● ACTIVE" mark beside tag, blue tag bg |
| Archived build row | row.isArchived | Tag color muted, coverage bar full grey |
| Coverage hover | Hover bar | Tooltip "78% of users (347 of 445)" |
| Regression hint | errorsDelta > 0 | Red "↑ +5 vs v1.4.1" inline |
| Deploy a comparison | Click Compare | Comparison tab opens; user picks right-hand build from list dialog |
| Rollback flow | Click Rollback | Confirmation: "Make {tag} active again? This re-points the default policy." |
| Empty branch | branch has no builds | "No builds on {branch} yet. Cut a build from the Designer to deploy." |

## Copy

- Page title: "Builds"
- Help icon: "Immutable, published app snapshots. Each shows what it serves and how it compares."
- Active mark: "● ACTIVE"
- Coverage label active: "{percent}% of traffic"
- Coverage label dim: "{userCount} / {totalUsers} users" OR "role: {roleName}"
- Live stats keys: "Runs (1h)", "Errors", "Avg quality", "Incidents"
- Delta tooltips: "{delta} vs previous build on this branch"
- Detail crumb: "Build"
- Detail tabs: "Deployment", "Comparison", "Live runs", "Workflows", "Changelog"
- Deployment tab subtitle: "Who it's serving"
- Not-on-this-build footer: "Not on this build: {roleName} role → {tag} · {roleName} role → {tag}"
- Comparison tab cells: "Throughput", "p95 latency", "Error rate"
- Doc banner (Deployment): "What's a deployment view? Builds are artifacts; deployments are who's running them. This tab shows the slice of traffic each policy rule sends to this build."
- Rollback confirmation: "Make {tag} active again? This re-points the default policy back to {tag}."
- Empty branch: "No builds on {branch} yet. Cut a build from the Designer to deploy."

## Tokens

- Tag (selected/active build): `color/blue/500` bg, white text
- Tag (regular): `color/surface/alt` bg, `color/text/default` text
- Branch pill: white bg, `color/text/muted` text, `color/border/strong` border
- Active mark: `color/green/500` dot + green text
- Coverage bar segments: active `color/blue/500`, secondary `color/purple/500`, tertiary `color/border/strong`
- Cmp-cell up delta: `color/green/700`
- Cmp-cell down delta: `color/red/700`

## Open questions for engineering

- Coverage computation strategy (sampling vs exact).
- Refresh frequency.
- Comparison data source (reuse Live Evals BuildComparison entity?).
- Rollback API contract.
- Archive policy (when is a build archived?).

## Figma + user testing — pending

- [ ] Build hi-fi static mockups: Figma frame "Builds / List" with branch tabs (All · main · staging · beta), 4 rows (one selected active build with coverage 78%, one minor traffic on legacy role 15%, one beta 7%, one archived 0%), live stats with deltas.
- [ ] Build hi-fi static mockup: Detail — Deployment tab frame with "Who it's serving" rows (default policy + 2 overrides) and "Not on this build" footer line.
- [ ] Build hi-fi static mockup: Detail — Comparison tab frame with cmp-grid (3 cells: throughput / p95 / error rate) showing deltas plus scorer deltas below.
- [ ] Add Figma variants for the interaction states matrix.
- [ ] Run user test script: 5 tasks — (a) tell me which build is currently serving everyone, (b) which build is serving the legacy-tier role, (c) compare v1.4.2 to v1.4.1 and tell me which got worse, (d) find a live run on v1.4.2 and open it, (e) explain what "coverage" means here.
- [ ] Run with 5 participants (emphasize developer + ops mix).
- [ ] Append `## User test summary` section after testing concludes.
