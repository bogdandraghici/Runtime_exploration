# Runtime Prototype — Fix List

Gaps between `runtime-prototype.html` and the `runtime_starter.htm` spec. Ordered high → low signal. Tackle one at a time; items marked **[needs design]** require a brainstorming pass before patching.

---

## Tier 1 — High signal

### 1. Workflow Instances are not a navigable surface  ✅ **DONE**
**Spec:** §1 system map and §7 call out `WorkflowInstance` ("one end-to-end workflow run") and `Invocation` ("one AI-node execution") as first-class runtime entities. The starter is explicit that AI workflow runs are the same entity surfaced by Process Instances, UI Flow Sessions, and Live Evals — i.e. three lenses on one thing.
**Resolution:** added `MOCK_DATA.workflowRuns` with full per-node invocations and parent linkage. New `renderWorkflowRunDetail` right-slide panel surfaces the parent link in the header meta-row (the load-bearing cross-lens jump). Tabs: Trace / Variables / Scores. Surfaced in: Live Evals workflow page ("Recent runs" tier), new `#/live-evals/runs` cross-workflow route, instance detail "AI runs" tab (now real data), and Build detail "Workflow runs" tab. No new top-level nav entry — runs are reachable from every natural starting point.

### 2. Live Evals is significantly thinner than spec  ✅ **DONE**
**Spec:** §7 describes a multi-page Live Evals surface with five routes.
**Resolution:** Live Evals restructured into landing + per-workflow drill-in (workflow promoted to URL segment `#/live-evals/wf_xxx`). Workflow detail has tabs `[Overview] [Settings]` with page-level time window + Decisions facet filter. All six sub-items closed:
- **2a** Observation points + `DecisionsFacet` chip row (cycles values). Disabling an obs point in Settings removes its chip.
- **2b** Time window control (1h / 24h / 7d / Custom) drives all downstream sections.
- **2c** AI-node detail right-slide panel (Trace / Invocations / Scorers) — Tier 3 cards click to open.
- **2d** Settings tab with Scorer bindings table (sampleRate, dailyTokenBudget, threshold, enabled toggle) + Observation points table.
- **2e** Inline SVG time-series chart per scorer with threshold line + build-deploy markers.
- **2f** Tier 1 expanded to 8 metrics + terminal-outcome distribution + top-5 path classes.

The IA restructure (workflow → URL segment) was discovered mid-implementation and addresses a scope-ambiguity friction not in the original starter doc.

### 3. Build comparison is a stub  ✅ **DONE**
**Spec:** §2.4 + §7 describe a dedicated comparison page (`…/builds/:leftBuildId/compare/:rightBuildId`) that computes per-scorer deltas, per-AI-node deltas, decision-distribution shifts per observation point, and returns `'incompatible'` when builds share no scorers.
**Resolution:** new workflow-scoped comparison page at `#/live-evals/wf_xxx/builds/<l>/compare/<r>` (matches spec route shape). Build picker header with two `<select>` dropdowns + Swap. Four sections: aggregate deltas (8 metrics) · per-scorer with regression flags · per-AI-node latency/cost shifts · decision-distribution shifts (bar pairs per observation-point value). Archived builds (no scorer history) trigger an `incompatible` empty state.
**Entry points:**
- Workflow detail header → `Compare builds →` link (auto-defaults to active vs most-recent prior on same branch)
- Build detail header → `Compare to…` button (auto-picks a workflow with runs on this build)
- Build detail → `Comparison` tab (lists workflows with runs on this build, each links into that workflow's comparison page)

### 4. Build → "workflow runs scoped to this build" view is missing  ✅ **DONE**
**Spec:** §3 / §8 — `runtime/builds-list` has a `workflow-instances` child outlet for inspecting workflow runs that happened against a given build.
**Resolution:** added a "Workflow runs" tab to Build detail (between "Live runs" and "Workflows"), filtering `MOCK_DATA.workflowRuns` by `buildTag`. Tab count badge turns red when any run on the build has a breached scorer. Rows open the workflow run detail panel; overflow links to `#/live-evals/runs`. Adapted from spec's "child outlet" pattern to the prototype's right-slide-panel idiom.

---

## Tier 2 — Medium signal

### 5. Process Instance detail tabs are incomplete
**Spec:** §5 — eight tabs: Variables, Tokens, Subprocesses, Exceptions, Workflows, Trigger, Notifications, Audit Log.
**Prototype state:** four to six tabs: Overview, Variables, Activity tree, AI runs, [Incidents if any], Audit.
**Missing pieces:**
- **5a.** Subprocesses — Activity tree shows "No subprocesses for this instance" (line 3134); mock data has no sub-process tree (only `spawnedBy` in the other direction).
- **5b.** Trigger tab — initiator info not surfaced.
- **5c.** Notifications tab — outgoing notifications surface absent.
- **5d.** Token detail — Activity tree mentions tokens but doesn't surface `state` (ACTIVE/ON_HOLD/etc.), `nodesActionStates[]` per-node history, or `backSeq` navigation.
- **5e.** Variables edit affordance gated on `wks_process_instance_variables_edit` — variables are read-only.

### 6. Trigger and Task detail panels are stubs
**Spec:** §8 — Triggers and Task Manager are first-class surroundings; tasks come from Process Instances reaching human-task nodes.
**Prototype state:** `renderTriggerDetail` (line 3396) and `renderTaskDetail` (line 3399) return literal "Detail content coming in Task 8" placeholders.
**What "fixed" means:** flesh out both detail panels — Trigger needs schedule/event config, recent fires, spawned instances; Task needs source instance link, claim/complete actions, variables snapshot.

### 7. Overrides have Edit/Delete buttons but no working flow
**Spec:** §4 — overrides have full CRUD plus audit metadata (`createdBy`, `createdDate`, `modifiedBy`, `modifiedDate`).
**Prototype state:** Edit/Delete are decorative links (lines 2526, 2536). No modal, no audit metadata shown.

### 8. Param-axis resolver lacks an ordered resolution path
**Spec:** §4 — `resolutionPath` is an ordered list of `{description, matched}` showing which override rules were checked and which won.
**Prototype state:** Build axis correctly produces this (lines 2694-2722). Param axis is flattened per key — annotates `via/viaTarget/viaPriority` but no ordered path of evaluated rules (lines 2724-2757).

### 9. Audit Log is minimal
**Spec:** §5 — "full timeline" with token arrivals, finishes, action data per node.
**Prototype state:** instance detail audit shows just `started` + optionally `failed` or `completed` (lines 3087-3095).

---

## Tier 3 — Low signal / cosmetic

### 10. Permission gates not modeled
Starter mentions `wks_process_instance_variables_edit`, `ui_flow_session_read`, `wks_tasks_read`, `wks_live_evals_read`, `wks_active_policy_read`, `wks_config_param_overrides_read`. Prototype has no permission layer. Fine for prototype; flag if the redesign target is permission-aware.

### 11. Failed-fires drill-down goes back to trigger config, not a per-fire page
Spec §8 mentions a `runtime/failed-triggers/:id` per-fire detail. Prototype's failed-fires rows route to the parent trigger detail (line 2858). Incidents cover the failure context, so this may not need its own surface — design call.

### 12. `buildId` (canonical id) never exposed
Only `buildTag` shown in UI. Ops users may need the canonical `b_v142`-style id. Minor.

### 13. Scorer type coverage is narrow
Spec §7 lists scorer types covering reference-free quality (coherence, helpfulness, …) and runtime-only operational types (toxicity, schema-conformance, bias/PII, …). Prototype mock data uses only coherence / helpfulness / toxicity.

---

## How we'll work this

- Items marked **[needs design]** start with a brainstorm before any code.
- Other items are direct patches.
- Update this file as items move to in-progress / done.
