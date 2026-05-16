# Runtime Tab Redesign — Plan Index

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans to implement these plans task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Drive the Runtime tab redesign from spec to engineering-ready specifications, page by page.

**Architecture:** Per-surface decomposition. Each sub-plan is a self-contained design-execution track that produces an engineering-ready spec (information model, component API, hi-fi mockups, interaction states, copy, design tokens, user-test results, final handoff doc). Plans are sequenced by dependency, not by personal preference.

**Tech Stack:** Design tooling (Figma), FlowX Design System tokens, design specifications written as Markdown. No application code is produced by these plans — they end at engineering handoff.

**Source spec:** `docs/superpowers/specs/2026-05-15-runtime-redesign-design.md`
**Before/After companion:** `runtime_before_after.html`

---

## Sub-plans and sequence

| # | Plan | File | Phase | Blocks |
|---|------|------|-------|--------|
| 01 | **Shared detail-panel component** | [01-detail-panel.md](2026-05-15-runtime-redesign-01-detail-panel.md) | Foundation | 03, 04, 05, 06, 09 |
| 02 | **IA & sidebar nav** | [02-ia-sidebar.md](2026-05-15-runtime-redesign-02-ia-sidebar.md) | Foundation | 03–10 |
| 03 | **Overview dashboard** | [03-overview.md](2026-05-15-runtime-redesign-03-overview.md) | Page | — |
| 04 | **Instances page** | [04-instances.md](2026-05-15-runtime-redesign-04-instances.md) | Page | — |
| 05 | **Incidents page** | [05-incidents.md](2026-05-15-runtime-redesign-05-incidents.md) | Page | — |
| 06 | **Builds page** | [06-builds.md](2026-05-15-runtime-redesign-06-builds.md) | Page | — |
| 07 | **Configuration page** | [07-configuration.md](2026-05-15-runtime-redesign-07-configuration.md) | Page | — |
| 08 | **Live Evals integration** | [08-live-evals.md](2026-05-15-runtime-redesign-08-live-evals.md) | Page | — |
| 09 | **Triggers page** | [09-triggers.md](2026-05-15-runtime-redesign-09-triggers.md) | Page | — |
| 10 | **Tasks page** | [10-tasks.md](2026-05-15-runtime-redesign-10-tasks.md) | Page | — |
| 11 | **Documentation layer** | [11-docs-layer.md](2026-05-15-runtime-redesign-11-docs-layer.md) | Cross-cutting | — |

## Recommended execution order

1. **Phase A — Foundations** (must complete first, in order): 01 Detail panel → 02 IA & sidebar.
2. **Phase B — Pages** (can be parallelized): 03 Overview, 04 Instances, 05 Incidents, 06 Builds, 07 Configuration.
3. **Phase C — Integration & remaining** (after Phase B mid-point): 08 Live Evals, 09 Triggers, 10 Tasks.
4. **Phase D — Cross-cutting** (in parallel with Phase B/C from start): 11 Documentation layer.

## Standard task template (used in every sub-plan)

Every page/component plan follows the same 8-task scaffold so they're predictable and parallelizable.

1. Define information model — fields, types, defaults, validation rules.
2. Define component API — props, events, slots, behaviors, deep-link routes.
3. Build hi-fi static mockup in Figma — default state per breakpoint.
4. Define interaction states — hover, focus, active, disabled, loading, error, empty, alert.
5. Specify copy — labels, helper text, error messages, empty-state copy, tooltips.
6. Identify design tokens — colors, spacing, typography per FlowX DS.
7. User test — script + 3–5 participants from each target persona.
8. Iterate + handoff — final spec doc ready for engineering, including known open questions.

## Handoff format

Each sub-plan ends with a **Handoff deliverable** — a single Markdown doc in `docs/superpowers/specs/components/` or `docs/superpowers/specs/pages/` containing:

- Information model (with TypeScript-style interface as documentation)
- Component API
- Mockup links (Figma frame URLs)
- Interaction state matrix
- Final copy
- Token list
- User-test summary
- Open questions for engineering

Engineering picks up from there.
