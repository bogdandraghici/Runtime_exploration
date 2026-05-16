# Live Evals Integration — Plan 08

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Live Evals' internal structure (three-tier workflow page, AI nodes, comparison) is preserved. This plan specifies the three new inbound integration points and resolves the permission piggyback. Resolves friction #3 and #9 (with Plan 04), supports Plan 06 (Builds Comparison) and Plan 05 (AI quality Incidents).

**Architecture:** No structural change to Live Evals' three-tier workflow page. New: an embeddable "AI runs slice" component consumed by Instance details; a projection from `LiveAlertState` → Incidents; inline comparison data consumed by Builds detail. Permission `wks_live_evals_read` becomes the proper gate.

**Deliverables:**
- Modify (spec doc): `docs/superpowers/specs/pages/live-evals.md` — describes integrations
- Create: `docs/superpowers/specs/components/ai-runs-slice.md`
- Figma: "Live Evals / AI runs slice (embedded)"

**Depends on:** Plan 01, Plan 02. Coordinated with Plan 04, Plan 05, Plan 06.

---

### Task 1: Define information model for embedded slice

- [ ] **Step 1: Document data**

```markdown
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
```

- [ ] **Step 2: Commit**: `spec: ai-runs-slice information model`

---

### Task 2: Define embedded slice component API

- [ ] **Step 1: Document props**

```markdown
## Component API

Props: scope: LiveEvalsSlice.scope; runs: AiRun[]; totalCount: number; onRunClick(id); onDeepLinkOut(); maxRows?: number = 5.

Render: card with violet (`color/purple`) palette to signal Live Evals origin. Lists up to maxRows; "Open in Live Evals →" link in the head deep-links to `/runtime/live-evals/{workflowId}?instance={instanceId}` (or build/workflow scope variant).

Scorer chip color: green when score above threshold or no threshold, red bold when breached, grey "—" when not scored.

Empty state: "No AI runs for this {scope.kind} yet."
```

- [ ] **Step 2: Commit**: `spec: ai-runs-slice component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: Figma frame**

"Live Evals / AI runs slice (embedded)" — three example rows (classifier · LLM call with breached coherence · embedding) within the violet card with deep-link CTA in the head.

- [ ] **Step 2: Commit**: `spec(ai-runs-slice): embedded slice mockup`

---

### Task 4: Define LiveAlertState → Incident projection

- [ ] **Step 1: Document the projection rules**

```markdown
## LiveAlertState → Incident projection

When a `LiveAlertState` record exists with `status: 'breached'`:
- Project to IncidentRow with stage `'ai-quality'`
- severity: derive from breach magnitude — critical if currentValue is more than 2x past threshold delta, high if more than 1x, medium if just crossed, low if borderline (configurable thresholds)
- status: open while breached; auto-resolve when alert clears for N minutes (default 10, configurable per binding)
- sourceEntity: { kind: 'scorer', id: bindingId }
- title: "{scorerName} scorer breached — {workflowName} / {nodeName}"
- subline: "value {currentValue} · threshold {threshold} · {affectedRunsCount} affected runs in last hour"

The Live Evals page still owns the canonical scorer + alert configuration. The Incidents page is a read-side projection — clicking a row deep-links into Live Evals for scorer config edits.
```

- [ ] **Step 2: Commit**: `spec: livealertstate-to-incident projection`

---

### Task 5: Document Builds Comparison consumption

- [ ] **Step 1: Document the integration**

```markdown
## Builds Comparison integration

The Builds page Comparison tab (Plan 06) consumes the existing `BuildComparison` entity from Live Evals' comparison endpoint. No new endpoint required. The render is the cmp-grid (throughput/p95/error-rate) + scorer-delta rows.

Deep-link reverse: from Builds Comparison tab a "Open full comparison in Live Evals" button takes the user to the existing comparison page `/runtime/live-evals/{workflowId}/builds/{leftId}/compare/{rightId}`.
```

- [ ] **Step 2: Commit**: `spec: builds-comparison integration`

---

### Task 6: Resolve permission piggyback

- [ ] **Step 1: Document permission migration plan**

```markdown
## Permission migration

Today: `wks_builds_read` gates the Live Evals route — temporary piggyback because `wks_live_evals_read` exists in `PermissionValue` enum but is not granted.

Migration:
1. Backend grants `wks_live_evals_read` to all roles currently granted `wks_builds_read` (zero-disruption).
2. Frontend swaps the route guard from `wks_builds_read` → `wks_live_evals_read`.
3. Future: `wks_builds_read` continues to gate the Builds page; the two are now independent.

Side effect: the inbound integrations (AI runs slice in Instance detail, AI quality Incidents) each require `wks_live_evals_read` to render. If a user lacks it, the slice renders an empty state with a "no permission" hint.
```

- [ ] **Step 2: Commit**: `spec: live-evals permission migration`

---

### Task 7: User test the embedded slice and projections

- [ ] **Step 1: Script**

4 tasks: (a) open an Instance detail and find its AI run quality, (b) deep-link from a slice into Live Evals, (c) find an AI quality incident and explain where its data comes from, (d) view a Build's comparison and explain the relationship to Live Evals.

- [ ] **Step 2: Run with 5 participants (mix of personas)**

- [ ] **Step 3: `## User test summary`**

- [ ] **Step 4: Commit**: `spec(live-evals): integration user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: `## Open questions for engineering`**

Items: API: does the slice need a dedicated endpoint or can it reuse the existing instance-workflows query? auto-resolve threshold default, scorer severity mapping function, permission migration timing (release-coupled to UI), backwards-compat shim.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(live-evals): handoff`
