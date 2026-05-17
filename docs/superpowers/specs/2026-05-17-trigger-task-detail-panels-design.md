# Trigger + Task Detail Panels — Design

> **Status:** Draft, 2026-05-17. Closes fix-list Tier 2 #6.
> **Companions:** [Triggers page spec](pages/triggers.md), [Tasks page spec](pages/tasks.md), [fix-list.md](../../../fix-list.md)

## Context

Two right-slide detail panels in the prototype are currently stubs that render literal "Detail content coming in Task 8" placeholders:

- `renderTriggerDetail` (`runtime-prototype.html` ~line 5077)
- `renderTaskDetail` (~line 5080)

Both are reachable today — Trigger from the Triggers list and from Incident detail's `sourceEntity.kind === 'trigger'` path; Task from the Tasks list. After Tier 2 #5 the Trigger panel is also reachable from the Instance Overview's `Trigger` KV row when an initiator has a `triggerId`. The stub state is now the single biggest jagged edge in the prototype.

This design fleshes both panels out using the same right-slide chrome the Instance / Incident / Build / WorkflowRun panels already use, with in-session mutation for Claim / Complete consistent with the Variables-edit pattern from item 5e.

## Design decisions

The choices below were settled in the brainstorming session preceding this doc.

### Trigger detail — three tabs

**Tabs:** Configuration · Recent fires · Failures (the Failures tab is conditional — only rendered when the trigger has ≥1 failed fire in `MOCK_DATA.triggerFires`).

**Header (always visible above tabs):**
- Crumb: "Trigger details" + monospace `id`.
- Title: `it.name`.
- Subtitle: kind icon (`⏱` cron / `↘` webhook / `✦` event) + kind label + status pill (active/paused/erroring) + a `last fire {when}` chip when defined.

**Configuration tab:**
- Doc banner: "What's a trigger? Something that starts a process — on a schedule (cron), an incoming event (webhook), or an internal signal. Spawned instances appear in Instances."
- KV block:
  - **Type** — icon + label
  - **Schedule** — mono when cron (`0 2 * * *`), human when not (`on event`, `webhook`)
  - **Spawns** — definition name + monospace `spawnsDefinitionId` (no link in this prototype — the definitions page doesn't exist as a navigable surface)
  - **Last fire** — `it.lastFire`
  - **Status** — pill with current state; tooltip explains
  - **Errors (24h)** — count, red when > 0
  - **Audit** — `Created by {createdBy} on {createdDate}` / `Modified by {modifiedBy} on {modifiedDate}` rows

**Recent fires tab:**
- Heading "Recent fires" with a `${shown} of ${total}` pill (matching the existing pattern from Live Evals).
- Table rows from `MOCK_DATA.triggerFires.filter(f => f.triggerId === it.id)`, ordered newest first.
- Columns: When · Outcome · Spawned (click-through to instance for success) / Reason (for failed).
- Empty state: "No fires recorded for this trigger yet."

**Failures tab** (conditional):
- Same table, filtered to `outcome === 'failed'`.
- For failed fires that have a corresponding Incident (`MOCK_DATA.incidents` with matching trigger id in `sourceEntity`), the row has an "Open incident →" link.
- Tab badge shows failure count.

### Task detail — three tabs

**Tabs:** Overview · Form · History.

**Header:**
- Crumb: "Task" + monospace `id`.
- Title: `it.title`.
- Subtitle (meta-row): source instance chip (mono `instanceId` + definition name, fully clickable), priority pill, due chip (`overdue 2h` red / `in 4h` yellow / `due in 12h` default), completed pill when applicable.

**Overview tab:**
- Doc banner: "Tasks come from processes. When a Process Instance reaches a human-task node it creates a task here and waits for someone to complete it. Opening the originating instance shows where in the flow this is happening."
- Action button row at the top of the body:
  - Unassigned + not completed: `Claim` (primary)
  - Assigned to me + not completed: `Open form →` (primary, switches to Form tab) and `Reassign` (ghost, decorative for now)
  - Assigned to other + not completed: read-only "Assigned to {name}" with no action
  - Completed: read-only "Completed" pill
- KV block: Source instance (linkable), Assignee (avatar + name or "Unassigned"), Due, Priority, Created.

**Form tab:**
- 2–3 synthesized fields per task. Each task gets a `form: [{label, name, type, options?, placeholder?}]` mock. Field shapes:
  - `select` — labeled select with 2–4 options
  - `textarea` — labeled textarea, ~3 rows
  - `text` — labeled single-line input
- All fields are disabled when the task is completed.
- Submit button at the bottom: `Complete task` (primary). On click, marks completed (same as `completeTask(taskId)`) and switches to the History tab.
- Cancel button: ghost, closes the panel.
- Doc banner on first encounter: "Forms are mocked — fields shown here are illustrative and don't validate or persist."

**History tab:**
- Timeline using the existing `.timeline / .tl-row` chrome. Events synthesized per task:
  - Always: `created · from {instanceId}` at `task.createdAt` (synthesize if missing — derive from `dueAt` or `'recently'`)
  - When `assignee` is set or `taskMutations.assignee` is set: `claimed by {displayName}`
  - When `taskMutations.completed`: `completed by {me}` at "just now"
- Empty state: only the "created" event for fresh, untouched tasks.

### In-session mutations — `STATE.taskMutations`

New STATE field: `taskMutations: Record<taskId, { assignee?, completed? }>`.

- `claimTask(taskId)` — sets `mutations.assignee = { userId: 'me', displayName: 'AB' }`. The display name "AB" matches the existing "me" identity used in mock task `assignee` fields.
- `completeTask(taskId)` — sets `mutations.completed = true`. Switches the detail panel to the History tab (`STATE.detail.tab = <history-index>`) and re-renders.
- `renderTaskDetail` reads the merged effective task: `effective.assignee = mutations.assignee ?? task.assignee`; `effective.completed = mutations.completed || task.completed`.
- The Tasks list (`renderTasks`) reads the same merged view so that `Complete` correctly removes the row from the "pending" filter and shows it under "completed".
- Mutations persist for the page session (same contract as `variableEdits` — no clear on `closeDetail`).

### Mock-data additions

**`MOCK_DATA.triggers`** — extend each trigger with:

```
spawnsDefinitionId: 'def_<slug>',
audit: { createdBy: 'alice', createdDate: '14d ago', modifiedBy: 'bob', modifiedDate: '2d ago' },
```

Values are illustrative; not all triggers need distinct authors — keep one or two flavors.

**`MOCK_DATA.tasks`** — extend each task with:

```
createdAt: '12m ago' | '2h ago' | etc.,
form: [
  { label: 'Decision', name: 'decision', type: 'select', options: ['Approve', 'Reject', 'Send back'] },
  { label: 'Notes',    name: 'notes',    type: 'textarea', placeholder: 'Add reasoning…' },
],
history: [
  { at: '12m ago', kind: 'created', label: 'Task created from inst_55a1c0 · Loan application' },
  // claim/complete derive from mutations
],
```

Per-task form fields are synthesized to fit the title: e.g. the Loan task gets `Decision` + `Amount approved` + `Notes`; the KYC verification task gets `Identity verified` + `Notes`. Don't ship cookie-cutter forms — synthesize one tailored field per task.

## Visual conventions

- Reuse `.tabs / .tab.active` for the tab strip in both panels.
- Reuse `.kv` table for KV blocks.
- Reuse `.timeline / .tl-row` and `.tl-row .dot` for History.
- Reuse `.related-card` for Recent fires rows if they want a card look; otherwise keep a slim table (`<table>` with `.fr-table` styling like the friction list, or a simpler ad-hoc grid).
- Action buttons: `.btn.primary` / `.btn.ghost` (already in the file).
- New tab badge for "Failures" count: reuse the existing `.count.alert` pattern used elsewhere on the Instance Incidents tab.

## Out of scope

- Backend persistence — in-session only (`STATE.taskMutations` does not survive a page reload, same as `STATE.variableEdits`).
- Pause/resume toggle on Trigger panel — config mutation, deferred.
- Real form-rendering engine, validation, or schema-driven forms.
- Reassign affordance (button rendered as decorative ghost; no working flow).
- Deep links from Failures-tab rows to Incident detail when the failed fire isn't already in `MOCK_DATA.incidents` (gracefully omit the link).
- Inline-style → CSS-class sweep (file-wide pattern fight; still deferred).

## Acceptance

- Opening trigger `cron_retention` from the Triggers list renders three tabs (Configuration / Recent fires / Failures). Configuration shows schedule `0 2 * * *`, spawns `retention_v2`, audit lines. Recent fires shows two fires from mock data; Failures shows the failed cron fire and links to Incident `inc_006` if present.
- Opening trigger `cron_kpi` (which has no failures) renders only two tabs: Configuration and Recent fires.
- Opening trigger `webhook_sf_sync` from Incident `inc_003` renders correctly — the Incident panel's source-entity link reaches this panel.
- Opening Instance `inst_55a1c0` → Overview → clicking "Loans API webhook" opens the Trigger panel for `webhook_loans_api` (the new cross-link from Tier 2 #5).
- Opening task `task_1` (Approve loan application) → Overview → shows source instance link to `inst_55a1c0`, Claim button (since unassigned in mock — actually `task_1` is assigned to me; pick `task_2` for the unassigned demo). Clicking Claim updates assignee, button changes to "Open form →", switching to Form tab is implicit via the button.
- Opening task `task_2` (Verify identity documents, unassigned) → click Claim → assignee becomes "AB"; reopen → assigned to me, Open form button shows. Click Open form → Form tab, click "Complete task" → mutation flips completed, switches to History tab, history shows "claimed" + "completed" events. Task list "Pending" view no longer includes this task.
- All five surfaces preserve doc-banner dismissal via `STATE.dismissedBanners`.

## Risk surface

- **Trigger panel's source-entity reachability across Incident detail.** Verify by opening `inc_003`, then opening its source trigger from the panel's "Related" / source link. Currently the Incident "related" section doesn't pass `sourceEntity` to `openDetail` for triggers — confirm the Incident panel deep-links work or add the link if missing.
- **Task list re-render on Complete.** The Tasks list lives at `renderTasks(root)` and reads filters from `STATE.filters.tasks`. `completeTask` needs to invalidate it via `render()` so the row leaves the pending filter view. If the panel-only re-render leaves the underlying list stale, the user sees "pending" with the just-completed task still in it.
- **Header meta-row width** on Task panel. The source instance chip + priority pill + due chip can crowd narrow viewports. Spec-level — visual; will refine in implementation if it looks broken.
