# Runtime Tab Redesign — Design Spec

**Date:** 2026-05-15
**Scope:** Full redesign of the Runtime tab in the FlowX application header (sibling to Config — out of scope here).
**Companion:** [runtime_starter.htm](../../../runtime_starter.htm) — current-state anatomy and friction inventory used as input.

---

## 1. Goals

1. **Real insight into runtime at a glance.** A new Overview landing surfaces four insight zones — health & incidents, live activity, deployment status, AI quality & cost — within five seconds of opening the tab.
2. **Equal-weight support for three personas.** Ops on-call, app developer, and support/business analyst all use this tab. No persona is sacrificed for another; the landing exposes entry points for each.
3. **Consolidate the friction in the current anatomy.** All nine friction points listed in the starter doc are resolved or explicitly de-scoped (see §10).
4. **Self-teaching.** Any user — new hire, business analyst, sysadmin — should be able to learn every concept from inside the product, without external docs.

## 2. Non-goals

- Redesigning Config tab.
- Changing the BPMN process semantics or AI workflow execution model.
- Building a full ticketing / incident-management system inside Incidents (lightweight Ack / Resolve flow only).
- Adding new data sources or new types of telemetry. Everything shown today stays; nothing else is invented.

## 3. Information architecture

The Runtime tab presents an 8-entry left sidebar, semantically grouped:

```
Overview              (always default landing)

LIVE STATE
  Instances           [count] — merged Process Instances + UI Flow Sessions
  Incidents           [alert count] — merged Failed Process Start + Failed Triggers + Exceptions + AI breaches
  Live Evals          [warn count]

DEPLOYMENT
  Builds              [active build chip]
  Configuration       — merged Active Policy + Config Param Overrides
  Triggers            — combined Scheduled Processes + Manage Triggers + their failure history

WORK
  Tasks               [pending count]
```

Reduction: 11 entries today → 8 entries proposed. Three structural merges plus one new entry (Overview).

Sidebar items carry **live status badges** (incident counts, in-flight counts, active build chip) so the nav itself is a glanceable status surface before any click.

Footer of the sidebar: a persistent "All systems active" pulse and a "Tour & help" link — anchors the documentation layer.

## 4. Overview dashboard

Shape: **activity-first** with strong visual hierarchy.

| Region                     | Role                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------- |
| **Incident banner** (top, conditional) | Visible only when there are open incidents or breached scorers. One-line summary + Review action. Disappears when healthy. |
| **Activity feed** (left, ~65% width) | The protagonist. Real-time stream of starts / finishes / exceptions / scorer breaches / deploys / trigger fires. Filter chips + search. Day-divider rhythm. Each row deep-links into its source. |
| **KPI stack** (right column) | Three subordinate cards stacked: In flight (instance + session + task counts), Deployment (active build + traffic coverage), AI quality (top scorer movements). |
| **Trends strip** (bottom)  | Peripheral: throughput, p95 latency, errors/hr, cost/hr — sparkline + value each. |

Every card title carries a `?` affordance opening a definition popover.

## 5. Instances — merged surface

One list, type as a column. Process Instances and UI Flow Sessions share chrome.

**List columns:** Type (green P icon for process, purple F icon for UI flow) · Identity (ID + definition name) · Status (normalized) · Build · At step (current node / state info) · Started.

**Filters:** Type · Status · Build · Definition · Variable (key=value) · search by ID/variable · time window.

**Normalized status model** (resolves friction #7). Five canonical states:

| Canonical | Source — Process | Source — UI Flow |
|---|---|---|
| Running    | `STARTED`                                      | active session |
| Waiting    | `ON_HOLD`                                      | session waiting on user input |
| Completed  | `FINISHED`                                     | session completed |
| Failed     | `FINISHED_WITH_ERROR`, `FAILED`                | session errored |
| Terminated | `ABORTED`, `EXPIRED`, `TERMINATED`, `DISMISSED` | session ended early |

Raw underlying state shown on hover of the pill (state pill tooltip — see §11).

**Cross-linking:** Process Instance rows spawned by a UI Flow Session carry an inline "↑ spawned by sess_xxxx" indicator with a click-through. The Session's Activity tree tab includes the spawned process as a child.

**Detail panel** opens as a right-slide panel. Tabs (normalized for both types):

- Overview — KPIs + status + identity
- Variables
- **Activity tree** — type-adaptive: for processes, shows tokens + subprocesses; for sessions, shows spawned processes. Same metaphor, different content.
- **AI runs** — embedded Live Evals slice showing workflow runs for this instance/session, with scorers if scored. "Open in Live Evals →" deep-link.
- Incidents — only shown if there are any
- Audit

This resolves friction #2 (symmetric concepts), #3 and #9 (workflow runs only have one canonical surface — Live Evals — surfaced everywhere they're useful), #8 (one shared detail-panel component).

## 6. Incidents — unified "what's broken"

Four-stage taxonomy, one list (resolves friction #5):

| Stage          | Source                                              |
|----------------|-----------------------------------------------------|
| Pre-start      | Failed Process Start                                |
| Trigger        | Failed Triggers                                     |
| Mid-execution  | Exceptions (active incidents on Process Instances)  |
| AI quality     | LiveAlertState records with `status: breached`      |

Stage tabs at the top act as the primary filter. AI quality breaches are surfaced here because, from an ops perspective, a breached scorer IS an incident — but Live Evals remains the canonical config + time-series home.

**Row anatomy:** severity bar (4px left band: crit / high / med / low) · title with stage pill · sub-line with source IDs + brief context · Build chip · time · status pill (Open / Ack / Resolved).

**Detail panel:** Context (root cause, exception type, location, recurrence) · Stack trace · Timeline (merges exception events + token movements + build deploys for correlation) · Similar (related past incidents).

**Related entities panel** in the detail shows upward (parent UI flow session), downward (failing instance), and sideways (the Build that introduced the regression). This is the cross-navigation friction #3 and #4 needed.

**Lightweight workflow:** Acknowledge · Assign to me · Resolve. Three states, no escalation tree.

## 7. Builds — artifact list + deployment lens

Resolves friction #6.

**List columns** per build row: Tag · Branch · Active mark · Cut metadata · **Coverage bar with %** of traffic · Live stats (runs in last hour, errors with delta vs previous build, average AI quality with delta, open incident count).

Coverage bar is the new affordance — answers "who's running this build right now?" without bouncing to Configuration.

**Branch tabs** filter by source branch (All · main · staging · beta).

**Detail panel tabs:**

- **Deployment** — Who it's serving (default + each override rule's slice with %) and who it's *not* serving (the fallback routing for non-matching callers).
- **Comparison** — Pulled from Live Evals' existing BuildComparison engine. Throughput / p95 / error-rate deltas + per-scorer deltas vs the previous build on this branch.
- **Live runs** — Embedded slice of current Instances running on this build.
- **Workflows** — Definitions in this build.
- **Changelog** — Workflows / processes / params changed since the previous build.

Actions: Compare to (any build) · View in Live Evals · Rollback.

## 8. Configuration — two axes, one resolver

Resolves friction #1.

Two axes co-located on one page:

- **Build version** (top section) — default policy (LATEST_ON_BRANCH or FIXED_VERSION) + override table (USER/ROLE · priority · enabled).
- **Config parameters** (bottom section) — defaults map + override table (same grammar, but value is a params map).

Both sections share **identical override grammar:** target type (USER/ROLE) · target value · effective payload · priority · enabled toggle · audit.

**Persistent right-side resolver** is the new affordance the merger unlocks:

- Always-visible "Test as user" form: username + roles input.
- Resolves *both* axes for the given identity, in one shot.
- Shows the **resolution path** for the build axis (ordered list of which override rules were checked, which matched, which was won by priority, which were skipped).
- Shows the **source per param** for the config axis (default vs override rule), including notes about disabled rules that would otherwise have matched ("beta-testers override would set 0.9 but is OFF").

This replaces today's modal `test-effective` dialog; the resolver becomes a continuous diagnostic surface, not a buried tool.

## 9. Live Evals · Triggers · Tasks

**Live Evals.** Three-tier workflow page (Telemetry / Workflow scorers / AI nodes) preserved as-is. New: explicit inbound integration points.

- Embedded "AI runs" slice on every Instance detail panel (deep-links into Live Evals).
- Breached scorers surface as AI quality Incidents.
- Build comparison data renders inline on the Builds page detail.

Permission piggyback noted in the starter (`wks_builds_read`) gets resolved as part of implementation; `wks_live_evals_read` becomes the proper gate.

**Triggers.** Combined surface with four tabs: Scheduled · External · Recent fires · Failed fires.

- Scheduled and External are the configuration views (today's Scheduled Processes + Manage Triggers).
- Recent fires shows the cross-trigger execution history.
- Failed fires shows trigger-stage failures (also double-surfaced in Incidents at the Trigger stage).

**Tasks.** Largely preserved — the user-task inbox concept is distinct enough to stand alone. Two changes:

- Every row shows the originating Instance ID with a click-through.
- The same task surfaces in the originating Instance's Activity tree tab — full cross-link.

## 10. Cross-cutting patterns

### 10.1 Shared detail-panel component

Resolves friction #8. One reusable component handles every entity detail panel — Instance, Incident, Build, Trigger, Task. Same chrome:

- Header with topline crumb, title, sub-line, meta-row of pills, action buttons.
- Horizontal tab strip with counts.
- Body of stacked cards (`.panel`) with `h4 + ?` headings.
- Optional yellow doc-banner at top when a tab/feature is novel.

Default placement: right-slide panel taking 460–480px. Maximizable to full route. Closeable.

### 10.2 Documentation layer

Six consistent affordances:

1. **Inline help icon** — next to every page title and panel heading. Opens a definition popover with a glossary link.
2. **Contextual doc banner** — yellow inline banner explaining a tab or feature on first encounter. Dismissable per-user; resurfaces only when the feature changes.
3. **Empty-state explanation** — every empty list explains *what would appear here* and *how it gets here*.
4. **Tour mode** — persistent "Tour & help" entry in the sidebar footer. Launches a guided overlay walkthrough of the current page. Auto-runs once on the first visit to Overview.
5. **Glossary side panel** — single source of truth for every concept (Build, Policy, Instance, Token, Scorer, Override, Live Evals, …). Reachable from every `?` popover.
6. **State pill tooltip** — hovering a normalized state pill reveals the raw underlying state. Bridges simplified UI vocabulary to engineering vocabulary.

These are familiar patterns, applied consistently. The cumulative effect is that any user can learn the system by using it.

### 10.3 Build-as-anchor

Build chips appear consistently across every surface that has a build context — Instance rows, Incident rows, Activity feed entries, detail panels. A single visual treatment (monospace tag in a blue chip) makes the build context universally legible without explanation.

## 11. Friction resolution map

| # | Friction (per starter §9) | Resolution |
|---|---|---|
| 1 | Active Policy + Config Param Overrides duplicate shape | Merged into Configuration (§8) |
| 2 | Process Instances + UI Flow Sessions near-symmetric | Merged into Instances (§5) |
| 3 | AI workflow runs in three places | Live Evals canonical; embedded slice everywhere else with deep-link out (§5, §7, §9) |
| 4 | No unified "what's running now" | Overview activity feed + merged Instances list (§4, §5) |
| 5 | Three "things went wrong" surfaces | Merged into Incidents (§6) |
| 6 | Builds has no deployment view | New Deployment tab + coverage column on rows (§7) |
| 7 | Inconsistent status models | Five-state normalization with raw on hover (§5, §10.2) |
| 8 | Detail panel pattern drifted three times | One shared component (§10.1) |
| 9 | Workflows tab is redundant Live Evals slice | Embedded Live Evals slice; one source of truth (§5, §9) |

## 12. Open questions

These need product input before or during implementation, not blocking the design itself:

- **Severity model for Incidents** — four-level (Critical / High / Medium / Low) is proposed. Real-world: how does severity get assigned? Auto-derived from stage + recurrence + scorer threshold delta, or manually settable?
- **Incidents lifecycle** — when does an incident auto-resolve? (Suggested: when the underlying condition clears for N minutes; also a manual Resolve action.)
- **Cross-environment scope** — the design assumes one application × one environment. Multi-env (prod / staging) selector likely lives at the app-bar level, but its scope (apply to whole sidebar? per-page?) needs settling.
- **Activity feed retention** — how far back does the Overview feed go? (Suggested: live + last 24h on landing; "Load earlier" loads more.)
- **Resolver "what changes if I" mode** — the persistent Configuration resolver currently shows the *current* effective values. Future addition: a "preview" mode that shows what would change if a proposed override were added.

## 13. Implementation considerations

These are signal for the eventual writing-plans pass — not constraints.

- The shared **detail-panel component** is the highest-leverage piece of frontend work; building it well unblocks everything else and prevents the drift that produced friction #8.
- The **normalized status model** can be done as a UI-layer mapping initially without touching the backend ProcessState / UIFlowSession.status fields; a backend rationalization is optional follow-on.
- The **activity feed** needs an event ingestion path — likely a thin aggregator over existing event streams (process lifecycle events, exception events, scorer alerts, build deploys, trigger fires). No new sources, just unified consumption.
- The **deployment-coverage column** on Builds needs a usage join against Active Policy resolutions. Coverage % can be sample-driven if exact computation is expensive.
- **Tour & help** can ship in incremental waves — start with Overview, then add one page at a time. Glossary is small enough to seed in one pass.

## 14. References

- [runtime_starter.htm](../../../runtime_starter.htm) — current state and friction inventory
- Brainstorm session screens: `.superpowers/brainstorm/53270-1778829762/content/` (landing-approaches, IA, overview, instances, incidents, builds, configuration, remaining-surfaces)
