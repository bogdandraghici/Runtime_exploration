# Live Evals — Integration Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

Live Evals' internal structure — the three-tier workflow page (Telemetry / Workflow scorers / AI nodes) — is preserved structurally by this redesign. Nothing in the canonical Live Evals surface changes shape; it remains the single source of truth for scorer configuration, alert bindings, AI node telemetry, and build comparison.

What this spec covers is the three new *inbound* integration points that allow other surfaces to consume Live Evals data without duplicating it: (1) an embeddable AI runs slice rendered inside Instance detail panels, (2) a read-side projection from `LiveAlertState` records into the unified Incidents list, and (3) inline consumption of the existing `BuildComparison` entity by the Builds detail panel's Comparison tab. Together these resolve friction #3 (AI workflow runs surfaced in three places) and #9 (the redundant Workflows tab) from the starter doc.

Additionally, this spec resolves the permission piggyback that today gates the Live Evals route on `wks_builds_read`: the proper gate `wks_live_evals_read` already exists in `PermissionValue` and is activated as part of this work.

## LiveAlertState → Incident projection

When a `LiveAlertState` record exists with `status: 'breached'`:
- Project to IncidentRow with stage `'ai-quality'`
- severity: derive from breach magnitude — critical if currentValue is more than 2x past threshold delta, high if more than 1x, medium if just crossed, low if borderline (configurable thresholds)
- status: open while breached; auto-resolve when alert clears for N minutes (default 10, configurable per binding)
- sourceEntity: { kind: 'scorer', id: bindingId }
- title: "{scorerName} scorer breached — {workflowName} / {nodeName}"
- subline: "value {currentValue} · threshold {threshold} · {affectedRunsCount} affected runs in last hour"

The Live Evals page still owns the canonical scorer + alert configuration. The Incidents page is a read-side projection — clicking a row deep-links into Live Evals for scorer config edits.

## Builds Comparison integration

The Builds page Comparison tab (Plan 06) consumes the existing `BuildComparison` entity from Live Evals' comparison endpoint. No new endpoint required. The render is the cmp-grid (throughput/p95/error-rate) + scorer-delta rows.

Deep-link reverse: from Builds Comparison tab a "Open full comparison in Live Evals" button takes the user to the existing comparison page `/runtime/live-evals/{workflowId}/builds/{leftId}/compare/{rightId}`.

## Permission migration

Today: `wks_builds_read` gates the Live Evals route — temporary piggyback because `wks_live_evals_read` exists in `PermissionValue` enum but is not granted.

Migration:
1. Backend grants `wks_live_evals_read` to all roles currently granted `wks_builds_read` (zero-disruption).
2. Frontend swaps the route guard from `wks_builds_read` → `wks_live_evals_read`.
3. Future: `wks_builds_read` continues to gate the Builds page; the two are now independent.

Side effect: the inbound integrations (AI runs slice in Instance detail, AI quality Incidents) each require `wks_live_evals_read` to render. If a user lacks it, the slice renders an empty state with a "no permission" hint.

## Open questions for engineering

- Auto-resolve threshold default for the `LiveAlertState → Incident` projection — confirm 10 minutes as the cross-team default, or set per-binding only.
- Scorer severity mapping function — formalize the critical/high/medium/low thresholds and whether they're tunable per-scorer.
- Permission migration timing — is the backend grant of `wks_live_evals_read` release-coupled to the UI swap, or staged ahead?
- Backwards-compat shim — should the old `wks_builds_read` route guard continue to function during a transition window?

## Figma + user testing — pending

- [ ] Figma frame "Live Evals / AI runs slice (embedded)" — three example rows (classifier · LLM call with breached coherence · embedding) within the violet card with deep-link CTA in the head.
- [ ] User testing script — 4 tasks: (a) open an Instance detail and find its AI run quality, (b) deep-link from a slice into Live Evals, (c) find an AI quality incident and explain where its data comes from, (d) view a Build's comparison and explain the relationship to Live Evals.
- [ ] Run with 5 participants (mix of personas).
- [ ] Capture user-test summary and iterate.
