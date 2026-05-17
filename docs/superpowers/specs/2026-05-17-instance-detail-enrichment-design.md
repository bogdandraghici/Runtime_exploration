# Instance Detail Enrichment — Design

> **Status:** Draft, 2026-05-17. Closes fix-list Tier 2 #5 (sub-items 5a–5e).
> **Companion:** [Instances page spec](pages/instances.md), [fix-list.md](../../../fix-list.md)

## Context

The merged Instance detail panel today exposes 6 tabs (Overview · Variables · Activity tree · AI runs · Incidents · Audit) and resolves the Process Instance + UI Flow Session merge from friction #2. The starter spec's `§5` originally enumerated 8 tabs (Variables, Tokens, Subprocesses, Exceptions, Workflows, Trigger, Notifications, Audit). The redesign collapsed those, but a handful of starter-spec capabilities did not survive the merge cleanly and surface in the current prototype as missing data or stubs:

- Activity tree → Subprocesses section literally reads "No subprocesses for this instance" (no mock tree)
- No initiator info anywhere — Trigger tab was collapsed but its content didn't land elsewhere
- No outgoing notifications surface
- Tokens render as a single line with no state, no node action history, no back-seq navigation
- Variables are read-only with no edit affordance, despite a permission existing in the starter spec

This design closes those gaps without expanding the tab set or duplicating IA.

## Design decisions

The choices below were settled in the brainstorming session preceding this doc.

### 5a — Subprocess tree (in Activity tree)

**Decision:** Render a real recursive subprocess tree inside the existing "Subprocesses" section of `renderActivityTree`. Each row shows the type glyph (▸ for process, ◆ for uiflow), the definition name, the id, and a status pill. Clicking a row opens that child's instance detail. Replace the literal empty-state with: "This instance hasn't spawned any subprocesses."

**Data:** Add `subprocesses?: Array<{id, definitionName, status, type, atStep, subprocesses?}>` to a handful of representative `MOCK_DATA.instances` entries. The tree is shallow (≤2 levels deep) for demonstration; the renderer is recursive.

### 5b — Trigger initiator info (folded into Overview)

**Decision:** Surface initiator information in Overview, not as a separate tab. Existing "Spawned by" KV stays put (it covers parent-instance attribution). A new "Trigger" KV block sits below the Status block.

**Data:** Add `initiator: { kind: 'manual'|'cron'|'webhook'|'process'|'uiflow', triggerId?, triggerName?, userId?, when? }` to mock instances. Render in Overview:

- kind icon + label ("Cron · Nightly retention sweep", "Webhook · Stripe webhook", "Manual · u_4419", "Spawned · sess_a82c91", "Process · inst_19f3d4")
- `triggerName` linkable to `openDetail('trigger', triggerId)` if `triggerId` is set

### 5c — Notifications (woven into Audit timeline)

**Decision:** Notifications are not their own tab. They appear in the Audit timeline as additional rows, ordered chronologically with the existing lifecycle events.

**Data:** Add `notifications: Array<{at, channel: 'email'|'webhook'|'in-app', target, subject, status: 'sent'|'failed'}>` to mock instances. Audit renderer merges `notifications` with existing lifecycle events, sorts by `at`, and renders notification rows with:

- Envelope icon (sent) or warning icon (failed)
- Channel + target on the left, subject in the value column
- Failed rows use the existing `tl-row.bad` style

Doc banner: "**Audit timeline.** Lifecycle events (started, paused, completed) plus outgoing notifications (email, webhook, in-app) in one chronological log."

### 5d — Token detail (inline expand)

**Decision:** Each token row inside Activity tree's "Active tokens" section is independently expandable. The expansion reveals raw ProcessState (with hover tooltip), per-node action history, and a back-seq chip when applicable.

**Data:** Add `tokens?: Array<{id, name, state: ProcessState, stateDesc, currentNode, nodesActionStates: Array<{at, node, action, status}>, backSeq?: {node, reason}}>` to mock process instances. UI Flow sessions keep no `tokens` array.

UI behavior:

- Token row gets a chevron (▶/▼). Click toggles `STATE.expandedTokens` (a `Set<tokenId>`)
- Expanded body: raw state pill (reuses `statePill` pattern; hover shows `stateDesc`), then a vertical action-history list (timestamp · node · action · status), then a "Back to: {node}" chip if `backSeq` is set, with the back reason as a sub-line
- Collapsed default; expanded state is non-persistent (resets when the detail panel closes)

### 5e — Variables edit (visible button + working modal)

**Decision:** Variables tab shows a per-row pencil affordance. Edits are in-session only (no backend mock).

**Data + state:**

- New module state: `STATE.variableEdits: Record<instanceId, Record<key, newValue>>`
- Variables tab reads from `STATE.variableEdits[it.id]` first, falls back to `it.variables`
- Variable rows with an override display a small `Edited` badge next to the value

UI:

- Hover-revealed pencil icon on each row; click opens centered modal
- Modal has: read-only key (mono), current value, new value textarea (mono), Save + Cancel buttons, plus "Revert to original" link when the row already has an edit
- Save persists to `STATE.variableEdits`, closes modal, re-renders Variables tab
- Save is a no-op against backend; in-session only

Doc banner on Variables tab: "**Edit variables.** Inline edits go to a live running instance. Requires the **wks_process_instance_variables_edit** permission (always granted in this prototype)."

## Mock-data scope

Touches `MOCK_DATA.instances`. Not every instance gets every new field — only enough to demonstrate the surfaces are populated when relevant:

| Instance | subprocesses | initiator | notifications | tokens |
|---|---|---|---|---|
| `sess_a82c91` Onboarding chat (uiflow) | ✓ spawns `inst_19f3d4`, `inst_91ab23` | ✓ manual (u_4419) | ✓ in-app welcome | — (uiflow) |
| `inst_19f3d4` KYC review (failed) | ✓ spawns `inst_doc_scan` | ✓ process (sess_a82c91) | ✓ failed email | ✓ 1 token at Document upload |
| `inst_55a1c0` Loan application (waiting) | — | ✓ webhook (webhook_loans_api) | ✓ sent task notif | ✓ 1 token on hold |
| `inst_a01b3e` Nightly retention sweep | ✓ spawns `inst_sweep_batch_1`, `inst_sweep_batch_2` | ✓ cron (cron_retention) | — | ✓ 2 tokens (terminated) |
| `inst_7c20fa` Document classifier (running) | — | ✓ webhook (webhook_stripe) | — | ✓ 2 tokens active |
| Others | none | basic initiator only | none | none |

A new mock instance `inst_doc_scan` is added as the grandchild needed to demonstrate the recursive tree.

## Visual conventions

- Subprocess tree row: same `.tree-node` chrome as the existing top-level token row, indented via existing `.tree-child` wrapper, recursive
- Initiator block in Overview: same `.kv` table chrome, with kind chip in the value cell
- Notification rows in Audit: existing `.tl-row` / `.tl-row.bad`, with `📧` glyph in the `.dot` slot
- Token expansion: chevron in row's leading slot; expanded panel uses a sub-`.panel-block` with reduced padding
- Variable edit pencil: 12×12 inline icon, `var(--text-dim)` default, `var(--blue-500)` on hover
- Variable edited badge: `.count` style pill, label "edited"
- Variables modal: new minimal modal scaffold — `.app-modal` centered overlay (~420px wide, white card, shadow, scrim using same backdrop blur as the detail panel), close-on-Esc, close-on-scrim-click; reusable for future modals

## Out of scope

- Backend integration for variable edits (in-session only; intentional)
- Modeling other permissions (`wks_process_instance_variables_edit` is hard-coded as granted)
- Editing the action history or back-seq (read-only surface)
- Adding Trigger / Notifications as top-level tabs (rejected in favor of fold-in)
- Notifications filtering or replay actions (read-only timeline)

## Acceptance

- Opening `sess_a82c91` shows two subprocesses in Activity tree; clicking each navigates to its detail
- Opening `inst_19f3d4` Overview shows `Trigger · Process · sess_a82c91 (Onboarding chat)`
- Opening `inst_19f3d4` Audit shows the failed email notification interleaved chronologically with lifecycle events
- Opening `inst_19f3d4` Activity tree → clicking the token row reveals raw state `FINISHED_WITH_ERROR`, per-node action history, no back-seq
- Opening `inst_55a1c0` Variables → clicking the pencil on `amount` → editing → Save → row shows `42000` (or new value) with an `edited` badge; reopening the modal shows "Revert to original"
- All five surfaces preserve doc-banner dismissal via the existing `STATE.dismissedBanners` set
