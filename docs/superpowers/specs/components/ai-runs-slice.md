# AI Runs Slice — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The AI Runs Slice is an embeddable Live Evals component consumed inside Instance detail panels (and, with scope variants, by Builds and workflow contexts). It surfaces the most recent AI runs scoped to the host entity, with scorer status inline, and always provides a deep-link out to the canonical Live Evals workflow page.

This component is the cornerstone of the "Live Evals canonical, embedded everywhere else" pattern that resolves friction #3 and #9. Live Evals owns the data; consumers render a compact, read-only slice with a clear escape hatch back to the source of truth.

## Information model

interface LiveEvalsSlice {
  scope: { kind: 'instance'|'workflow'|'build'; id: string };
  runs: AiRun[];                  // capped to N most recent, configurable
  totalCount: number;
  hasScores: boolean;
}

interface AiRun {
  id: string;
  nodeName: string;
  nodeType: 'llm'|'embedding'|'classifier'|'retriever'|'other';
  inputSummary: string;
  latencyMs: number;
  cost?: number;
  scores?: { scorerName: string; value: number; threshold?: number; breached: boolean }[];
  ts: string;
}

## Component API

Props: scope: LiveEvalsSlice.scope; runs: AiRun[]; totalCount: number; onRunClick(id); onDeepLinkOut(); maxRows?: number = 5.

Render: card with violet (`color/purple`) palette to signal Live Evals origin. Lists up to maxRows; "Open in Live Evals →" link in the head deep-links to `/runtime/live-evals/{workflowId}?instance={instanceId}` (or build/workflow scope variant).

Scorer chip color: green when score above threshold or no threshold, red bold when breached, grey "—" when not scored.

Empty state: "No AI runs for this {scope.kind} yet."

## Open questions for engineering

- API: does the slice need a dedicated endpoint or can it reuse the existing instance-workflows query?
- `maxRows` default of 5 — is this the right default for the Instance detail context, or should it differ per scope kind?
- Deep-link routing for non-instance scopes (build, workflow) — confirm the URL grammar variants.

## Figma + user testing — pending

- [ ] Figma frame "Live Evals / AI runs slice (embedded)" — three example rows (classifier · LLM call with breached coherence · embedding) within the violet card with deep-link CTA in the head.
- [ ] User testing — script tasks covering finding AI run quality on an Instance detail and deep-linking out into Live Evals.
- [ ] Run with 5 participants (mix of personas) and capture summary.
