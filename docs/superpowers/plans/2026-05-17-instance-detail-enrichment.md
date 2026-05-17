# Instance Detail Enrichment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Close fix-list Tier 2 #5 (sub-items 5a–5e) by enriching the merged Instance detail panel with subprocess trees, trigger initiator info, outgoing notifications, expandable token detail, and a working Variables edit affordance.

**Architecture:** Pure additive edits to the single-file prototype `runtime-prototype.html`. New mock-data fields per spec, a small `STATE` extension for in-session variable edits + token expansion, a `.app-modal` scaffold, and renderer updates folded into `renderInstanceDetail` / `renderActivityTree`. The 6-tab Instance detail IA is preserved; Trigger info folds into Overview, Notifications fold into Audit.

**Tech Stack:** Single-file HTML/CSS/vanilla-JS prototype. No build step, no test framework. Verification is visual via a local browser open of `runtime-prototype.html`.

**Spec:** [`docs/superpowers/specs/2026-05-17-instance-detail-enrichment-design.md`](../specs/2026-05-17-instance-detail-enrichment-design.md)

---

## Conventions used in this plan

- Every code change is shown in full (the `old_string`/`new_string` pair for `Edit` calls).
- File paths are absolute from the prototype root: `runtime-prototype.html` is the only source file unless otherwise noted.
- Verification: open `runtime-prototype.html` in a browser (`open runtime-prototype.html` on macOS), then follow the linked action; the expected outcome is described per step.
- Commit boundaries are at the end of each task. Commits stay small and scoped.

---

## Task 1: Subprocess tree — data + recursive renderer

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.instances entries; `renderActivityTree` body)

**Closes:** Fix-list 5a

- [ ] **Step 1: Add subprocesses to representative instances + the new grandchild instance**

Use `Edit` on `runtime-prototype.html`. Target the existing line for `sess_a82c91`:

```
{ id: 'sess_a82c91', type: 'uiflow', definitionName: 'Onboarding chat', status: 'running', rawStatus: 'active', rawDesc: 'session in progress',
      buildTag: 'v1.4.2', atStep: 'Step 3 / 7 · Verify email', startedAt: '1m ago', user: 'u_4419', spawnedCount: 1,
      variables: { user_id: 'u_4419', plan: 'pro', step_count: 7, current_step: 3, locale: 'en-US' } },
```

Replace with:

```
{ id: 'sess_a82c91', type: 'uiflow', definitionName: 'Onboarding chat', status: 'running', rawStatus: 'active', rawDesc: 'session in progress',
      buildTag: 'v1.4.2', atStep: 'Step 3 / 7 · Verify email', startedAt: '1m ago', user: 'u_4419', spawnedCount: 1,
      variables: { user_id: 'u_4419', plan: 'pro', step_count: 7, current_step: 3, locale: 'en-US' },
      subprocesses: [
        { id: 'inst_19f3d4', definitionName: 'KYC review', type: 'process', status: 'failed', atStep: 'Document upload · 3 incidents' },
        { id: 'inst_91ab23', definitionName: 'Email verification', type: 'process', status: 'completed', atStep: 'Done' },
      ] },
```

Then target `inst_19f3d4`:

```
{ id: 'inst_19f3d4', type: 'process', definitionName: 'KYC review', status: 'failed', rawStatus: 'FINISHED_WITH_ERROR', rawDesc: 'raised exception during execution',
      buildTag: 'v1.4.2', atStep: 'Document upload · 3 incidents', startedAt: '3m ago',
      spawnedBy: { type: 'uiflow', id: 'sess_a82c91' }, activeIncidentCount: 3,
      variables: { applicant: 'u_4419', doc_type: 'passport', risk_score: 0.42 } },
```

Replace with:

```
{ id: 'inst_19f3d4', type: 'process', definitionName: 'KYC review', status: 'failed', rawStatus: 'FINISHED_WITH_ERROR', rawDesc: 'raised exception during execution',
      buildTag: 'v1.4.2', atStep: 'Document upload · 3 incidents', startedAt: '3m ago',
      spawnedBy: { type: 'uiflow', id: 'sess_a82c91' }, activeIncidentCount: 3,
      variables: { applicant: 'u_4419', doc_type: 'passport', risk_score: 0.42 },
      subprocesses: [
        { id: 'inst_doc_scan', definitionName: 'Document scan', type: 'process', status: 'failed', atStep: 'OCR · halted' },
      ] },
```

Then target `inst_a01b3e` Nightly retention sweep:

```
{ id: 'inst_a01b3e', type: 'process', definitionName: 'Nightly retention sweep', status: 'terminated', rawStatus: 'ABORTED', rawDesc: 'manually terminated',
      buildTag: 'v1.4.1', atStep: 'Manually aborted by ops', startedAt: '2h ago',
      variables: { sweep_id: 's_2031', processed: 412 } },
```

Replace with:

```
{ id: 'inst_a01b3e', type: 'process', definitionName: 'Nightly retention sweep', status: 'terminated', rawStatus: 'ABORTED', rawDesc: 'manually terminated',
      buildTag: 'v1.4.1', atStep: 'Manually aborted by ops', startedAt: '2h ago',
      variables: { sweep_id: 's_2031', processed: 412 },
      subprocesses: [
        { id: 'inst_sweep_b1', definitionName: 'Retention batch · users', type: 'process', status: 'terminated', atStep: 'aborted at parent' },
        { id: 'inst_sweep_b2', definitionName: 'Retention batch · logs', type: 'process', status: 'terminated', atStep: 'aborted at parent' },
      ] },
```

Finally, append `inst_doc_scan` (and the two retention batch stubs so click-throughs don't 404) at the **end of the `instances:` array**, before the closing `],`. Target the existing entry for `inst_d5e6f7`:

```
{ id: 'inst_d5e6f7', type: 'process', definitionName: 'Welcome email', status: 'failed', rawStatus: 'FAILED', rawDesc: 'unrecoverable failure',
      buildTag: 'v1.4.2', atStep: 'SMTP timeout', startedAt: '12h ago',
      variables: { user: 'u_9981', template: 'welcome_v3' } },
  ],
```

Replace with:

```
{ id: 'inst_d5e6f7', type: 'process', definitionName: 'Welcome email', status: 'failed', rawStatus: 'FAILED', rawDesc: 'unrecoverable failure',
      buildTag: 'v1.4.2', atStep: 'SMTP timeout', startedAt: '12h ago',
      variables: { user: 'u_9981', template: 'welcome_v3' } },
    { id: 'inst_doc_scan', type: 'process', definitionName: 'Document scan', status: 'failed', rawStatus: 'FINISHED_WITH_ERROR', rawDesc: 'subprocess halted',
      buildTag: 'v1.4.2', atStep: 'OCR · halted', startedAt: '3m ago',
      spawnedBy: { type: 'process', id: 'inst_19f3d4' },
      variables: { doc_id: 'd_9981', mime: 'application/pdf' } },
    { id: 'inst_sweep_b1', type: 'process', definitionName: 'Retention batch · users', status: 'terminated', rawStatus: 'ABORTED', rawDesc: 'aborted with parent',
      buildTag: 'v1.4.1', atStep: 'aborted at parent', startedAt: '2h ago',
      spawnedBy: { type: 'process', id: 'inst_a01b3e' },
      variables: { batch_id: 's_2031_users', processed: 218 } },
    { id: 'inst_sweep_b2', type: 'process', definitionName: 'Retention batch · logs', status: 'terminated', rawStatus: 'ABORTED', rawDesc: 'aborted with parent',
      buildTag: 'v1.4.1', atStep: 'aborted at parent', startedAt: '2h ago',
      spawnedBy: { type: 'process', id: 'inst_a01b3e' },
      variables: { batch_id: 's_2031_logs', processed: 194 } },
  ],
```

- [ ] **Step 2: Update `renderActivityTree` to render subprocesses recursively**

Find the existing `renderActivityTree(it)` function (around line 4099). Replace the entire function:

```js
function renderActivityTree(it) {
  if (it.type === 'uiflow') {
    const spawned = MOCK_DATA.instances.filter(x => x.spawnedBy && x.spawnedBy.id === it.id);
    return `${docBanner("What's an Activity tree?", "For UI Flow sessions, this lists the process instances the session spawned. For processes, it shows active tokens and subprocesses. Same idea, different content.", 'inst-activity-tree')}
      <div class="panel-block">
        ${panelHeading('Activity tree', 'UI Flow Session')}
        <div class="tree-node">
          <span style="color:var(--purple-500)">◆</span>
          <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
          <span class="pos">${it.status}</span>
        </div>
        ${spawned.map(s => `
          <div class="tree-child">
            <div class="tree-node ${s.status === 'failed' ? 'bad' : ''}">
              <span style="color:var(--green-500)">▸</span>
              <span class="lbl"><strong>${escapeHtml(s.definitionName)}</strong> · ${escapeHtml(s.id)}</span>
              <span class="pos">${s.status}${s.activeIncidentCount ? ` · ${s.activeIncidentCount} incidents` : ''}</span>
            </div>
          </div>`).join('') || '<div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">No spawned processes yet.</div>'}
      </div>`;
  } else {
    // Process: tokens + subprocesses
    return `${docBanner("What's an Activity tree?", "For processes, this shows active tokens (where execution pointers are right now) and any spawned sub-processes.", 'inst-activity-tree-p')}
      <div class="panel-block">
        ${panelHeading('Active tokens', 'Token')}
        <div class="tree-node ${it.status === 'failed' ? 'bad' : ''}">
          <span style="color:var(--green-500)">▸</span>
          <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
          <span class="pos">${it.status}${it.activeIncidentCount ? ` · ${it.activeIncidentCount} incidents` : ''}</span>
        </div>
      </div>
      <div class="panel-block">
        ${panelHeading('Subprocesses', null)}
        <div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">No subprocesses for this instance.</div>
      </div>`;
  }
}
```

With:

```js
function renderActivityTree(it) {
  const subprocessesHtml = renderSubprocessTree(it.subprocesses);
  if (it.type === 'uiflow') {
    return `${docBanner("What's an Activity tree?", "For UI Flow sessions, this lists the process instances the session spawned. For processes, it shows active tokens and subprocesses. Same idea, different content.", 'inst-activity-tree')}
      <div class="panel-block">
        ${panelHeading('Activity tree', 'UI Flow Session')}
        <div class="tree-node">
          <span style="color:var(--purple-500)">◆</span>
          <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
          <span class="pos">${it.status}</span>
        </div>
        ${subprocessesHtml || '<div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">This session hasn\'t spawned any processes yet.</div>'}
      </div>`;
  }
  // Process: tokens + subprocesses
  return `${docBanner("What's an Activity tree?", "For processes, this shows active tokens (where execution pointers are right now) and any spawned sub-processes.", 'inst-activity-tree-p')}
    <div class="panel-block">
      ${panelHeading('Active tokens', 'Token')}
      <div class="tree-node ${it.status === 'failed' ? 'bad' : ''}">
        <span style="color:var(--green-500)">▸</span>
        <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
        <span class="pos">${it.status}${it.activeIncidentCount ? ` · ${it.activeIncidentCount} incidents` : ''}</span>
      </div>
    </div>
    <div class="panel-block">
      ${panelHeading('Subprocesses', null)}
      ${subprocessesHtml || '<div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">This instance hasn\'t spawned any subprocesses.</div>'}
    </div>`;
}

function renderSubprocessTree(subs) {
  if (!subs || !subs.length) return '';
  return subs.map(s => {
    const glyphColor = s.type === 'uiflow' ? 'var(--purple-500)' : 'var(--green-500)';
    const glyph = s.type === 'uiflow' ? '◆' : '▸';
    // Look up the full instance to recurse on its own subprocesses (if present)
    const full = MOCK_DATA.instances.find(x => x.id === s.id);
    const nested = full && full.subprocesses ? renderSubprocessTree(full.subprocesses) : '';
    return `<div class="tree-child">
      <div class="tree-node ${s.status === 'failed' ? 'bad' : ''}" onclick="openDetail('instance','${s.id}')" style="cursor:pointer;">
        <span style="color:${glyphColor}">${glyph}</span>
        <span class="lbl"><strong>${escapeHtml(s.definitionName)}</strong> · <span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--text-muted);">${escapeHtml(s.id)}</span> · ${escapeHtml(s.atStep)}</span>
        <span class="pos">${escapeHtml(s.status)}</span>
      </div>
      ${nested}
    </div>`;
  }).join('');
}
```

- [ ] **Step 3: Verify in browser**

Open `runtime-prototype.html`. Navigate to Instances → click `sess_a82c91` Onboarding chat → Activity tree tab.
Expected: the section shows two child rows: KYC review (failed, red) and Email verification (completed). Clicking KYC review opens its detail panel.

Open `inst_19f3d4` KYC review → Activity tree → expect "Document scan" under Subprocesses.

Open `inst_a01b3e` Nightly retention sweep → Activity tree → expect two retention batches under Subprocesses, both terminated.

Open any other instance (e.g. `inst_55a1c0` Loan application) → expect empty-state copy "This instance hasn't spawned any subprocesses."

- [ ] **Step 4: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): render real subprocess tree in Activity tree (fix-list 5a)"
```

---

## Task 2: Trigger initiator — data + Overview block

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.instances entries; `renderInstanceDetail` Overview body)

**Closes:** Fix-list 5b

- [ ] **Step 1: Add `initiator` field to representative instances**

Target each of the listed instances and add an `initiator` property at the end of the object literal. Use multiple `Edit` calls. Example for `sess_a82c91` (this is the same line we touched in Task 1 — apply this edit *after* Task 1's edits land):

Existing (after Task 1):

```
      subprocesses: [
        { id: 'inst_19f3d4', definitionName: 'KYC review', type: 'process', status: 'failed', atStep: 'Document upload · 3 incidents' },
        { id: 'inst_91ab23', definitionName: 'Email verification', type: 'process', status: 'completed', atStep: 'Done' },
      ] },
```

Replace with:

```
      subprocesses: [
        { id: 'inst_19f3d4', definitionName: 'KYC review', type: 'process', status: 'failed', atStep: 'Document upload · 3 incidents' },
        { id: 'inst_91ab23', definitionName: 'Email verification', type: 'process', status: 'completed', atStep: 'Done' },
      ],
      initiator: { kind: 'manual', userId: 'u_4419', when: '1m ago' } },
```

Apply the same pattern for these instances (find each by id, append `initiator: { ... }` before the closing `}`):

| Instance id | initiator |
|---|---|
| `inst_19f3d4` | `{ kind: 'uiflow', triggerId: null, triggerName: 'Onboarding chat', parentId: 'sess_a82c91', when: '3m ago' }` |
| `inst_55a1c0` | `{ kind: 'webhook', triggerId: 'webhook_loans_api', triggerName: 'Loans API webhook', when: '12m ago' }` |
| `inst_a01b3e` | `{ kind: 'cron', triggerId: 'cron_retention', triggerName: 'Nightly retention sweep', when: '2h ago' }` |
| `inst_7c20fa` | `{ kind: 'webhook', triggerId: 'webhook_stripe', triggerName: 'Stripe webhook', when: '31m ago' }` |
| `inst_91ab23` | `{ kind: 'process', parentId: 'sess_a82c91', when: '1h ago' }` |
| `inst_doc_scan` | `{ kind: 'process', parentId: 'inst_19f3d4', when: '3m ago' }` |
| `inst_sweep_b1` | `{ kind: 'process', parentId: 'inst_a01b3e', when: '2h ago' }` |
| `inst_sweep_b2` | `{ kind: 'process', parentId: 'inst_a01b3e', when: '2h ago' }` |

For instances with no listed initiator, leave the field absent — Overview will render an em-dash.

- [ ] **Step 2: Add `renderInitiator` helper just above `renderInstanceDetail`**

Add immediately before `function renderInstanceDetail(it, tabIdx) {` (around line 4005):

```js
function renderInitiator(init) {
  if (!init) return '<span style="color:var(--text-muted)">—</span>';
  const kindLabels = { manual: 'Manual', cron: 'Cron', webhook: 'Webhook', process: 'Process', uiflow: 'UI Flow' };
  const kindIcons = { manual: '👤', cron: '⏱', webhook: '↘', process: '▸', uiflow: '◆' };
  const label = kindLabels[init.kind] || init.kind;
  const icon = kindIcons[init.kind] || '·';
  let body = '';
  if (init.triggerId && init.triggerName) {
    body = `<a style="color:var(--blue-500);cursor:pointer;" onclick="openDetail('trigger','${init.triggerId}')">${escapeHtml(init.triggerName)}</a>`;
  } else if (init.parentId) {
    body = `<a style="color:var(--blue-500);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11px;" onclick="openDetail('instance','${init.parentId}')">${escapeHtml(init.parentId)}</a>`;
  } else if (init.userId) {
    body = `<code style="font-family:'JetBrains Mono',monospace;font-size:11px;">${escapeHtml(init.userId)}</code>`;
  } else if (init.triggerName) {
    body = escapeHtml(init.triggerName);
  } else {
    body = '<span style="color:var(--text-muted)">—</span>';
  }
  const when = init.when ? `<span style="color:var(--text-dim);font-size:11px;margin-left:6px;">${escapeHtml(init.when)}</span>` : '';
  return `<span aria-hidden="true" style="display:inline-block;width:14px;color:var(--text-dim);">${icon}</span><strong>${escapeHtml(label)}</strong> · ${body}${when}`;
}
```

- [ ] **Step 3: Wire the Trigger row into the Overview KV**

Find the Overview body (line 4045 onward) and the existing `<div class="kv">` block. Target:

```
        <div class="kv">
          <span>Started</span><span>${escapeHtml(it.startedAt)}</span>
          <span>Updated</span><span>just now</span>
          <span>Definition</span><span>${escapeHtml(it.definitionName)}</span>
          <span>Build at start</span><span>${escapeHtml(it.buildTag)} (committed)</span>
          <span>Spawned by</span><span>${it.spawnedBy ? `<a style="color:var(--blue-500);cursor:pointer" onclick="openDetail('instance','${it.spawnedBy.id}')">${escapeHtml(it.spawnedBy.id)}</a>` : '<span style="color:var(--text-muted)">—</span>'}</span>
          <span>Spawned</span><span>${spawnedProcs.length ? spawnedProcs.map(p => `<a style="color:var(--blue-500);cursor:pointer" onclick="openDetail('instance','${p.id}')">${escapeHtml(p.id)}</a>`).join(', ') : '<span style="color:var(--text-muted)">—</span>'}</span>
        </div>
```

Replace with:

```
        <div class="kv">
          <span>Started</span><span>${escapeHtml(it.startedAt)}</span>
          <span>Updated</span><span>just now</span>
          <span>Definition</span><span>${escapeHtml(it.definitionName)}</span>
          <span>Build at start</span><span>${escapeHtml(it.buildTag)} (committed)</span>
          <span>Trigger</span><span>${renderInitiator(it.initiator)}</span>
          <span>Spawned by</span><span>${it.spawnedBy ? `<a style="color:var(--blue-500);cursor:pointer" onclick="openDetail('instance','${it.spawnedBy.id}')">${escapeHtml(it.spawnedBy.id)}</a>` : '<span style="color:var(--text-muted)">—</span>'}</span>
          <span>Spawned</span><span>${spawnedProcs.length ? spawnedProcs.map(p => `<a style="color:var(--blue-500);cursor:pointer" onclick="openDetail('instance','${p.id}')">${escapeHtml(p.id)}</a>`).join(', ') : '<span style="color:var(--text-muted)">—</span>'}</span>
        </div>
```

- [ ] **Step 4: Verify in browser**

- Open `inst_19f3d4` KYC review → Overview → expect `Trigger ◆ UI Flow · Onboarding chat 3m ago`. Click "Onboarding chat" — *but wait, there's no triggerId here; the parentId path renders sess_a82c91 instead*. Adjust expectation: the link text reads `sess_a82c91` (mono) and clicking it opens that instance.
- Open `inst_55a1c0` Loan application → expect `Trigger ↘ Webhook · Loans API webhook 12m ago`. Clicking the trigger name attempts to open trigger `webhook_loans_api` — that trigger lives in `MOCK_DATA.triggers` and will open in the trigger detail (still a stub until Task 6 of Tier 2 #6; expected to render the stub head).
- Open `inst_a01b3e` Nightly retention → expect `Trigger ⏱ Cron · Nightly retention sweep 2h ago`.
- Open `sess_a82c91` Onboarding chat → expect `Trigger 👤 Manual · u_4419 1m ago`.
- Open `inst_7c20fa` → expect `Webhook · Stripe webhook`.
- Open `inst_b73e1c` Payment retry (no initiator set) → expect `Trigger —`.

- [ ] **Step 5: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): surface trigger initiator in Instance Overview (fix-list 5b)"
```

---

## Task 3: Notifications — data + Audit interleave

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.instances entries; `renderInstanceDetail` Audit body; doc-banner copy)

**Closes:** Fix-list 5c

- [ ] **Step 1: Add `notifications` arrays to selected instances**

Append a `notifications` array to the following instances. For each, edit the closing `}` of the object literal and insert before it. Use the `Edit` tool with enough context to make the match unique. Show the field at the same indent level as `variables`.

| Instance id | notifications |
|---|---|
| `sess_a82c91` | `notifications: [ { at: '0m ago', atSortable: 0, channel: 'in-app', target: 'u_4419', subject: 'Welcome — let\'s finish setting up your account', status: 'sent' } ]` |
| `inst_19f3d4` | `notifications: [ { at: '4m ago', atSortable: 4, channel: 'email', target: 'compliance@flowx.ai', subject: 'KYC review escalated — Document scan failed', status: 'failed' }, { at: '8m ago', atSortable: 8, channel: 'in-app', target: 'u_4419', subject: 'Verification in progress', status: 'sent' } ]` |
| `inst_55a1c0` | `notifications: [ { at: '11m ago', atSortable: 11, channel: 'in-app', target: 'manager:approvals', subject: 'New loan task assigned — $42,000', status: 'sent' } ]` |
| `inst_d5e6f7` | `notifications: [ { at: '12h ago', atSortable: 720, channel: 'email', target: 'u_9981@example.com', subject: 'Welcome to FlowX', status: 'failed' } ]` |

`atSortable` is a numeric "minutes ago" used purely for chronological sort. Lower = more recent (matches the human label).

Example edit for `inst_19f3d4` (find the line we touched in Task 2):

Old:

```
      subprocesses: [
        { id: 'inst_doc_scan', definitionName: 'Document scan', type: 'process', status: 'failed', atStep: 'OCR · halted' },
      ],
      initiator: { kind: 'uiflow', triggerId: null, triggerName: 'Onboarding chat', parentId: 'sess_a82c91', when: '3m ago' } },
```

New:

```
      subprocesses: [
        { id: 'inst_doc_scan', definitionName: 'Document scan', type: 'process', status: 'failed', atStep: 'OCR · halted' },
      ],
      initiator: { kind: 'uiflow', triggerId: null, triggerName: 'Onboarding chat', parentId: 'sess_a82c91', when: '3m ago' },
      notifications: [
        { at: '4m ago', atSortable: 4, channel: 'email', target: 'compliance@flowx.ai', subject: 'KYC review escalated — Document scan failed', status: 'failed' },
        { at: '8m ago', atSortable: 8, channel: 'in-app', target: 'u_4419', subject: 'Verification in progress', status: 'sent' },
      ] },
```

Repeat for the other three.

- [ ] **Step 2: Add `parseStartedAtMinutes` helper and update the Audit renderer**

We need a tiny parser to convert `startedAt` strings like `"3m ago"`, `"2h ago"`, `"12h ago"` into a sortable number of minutes, so lifecycle events and notifications share a sortable axis.

Add this helper just above `renderInstanceDetail` (next to `renderInitiator` from Task 2):

```js
function parseAgoMinutes(s) {
  // Accepts "3m ago", "2h ago", "5d ago", "just now", "now"
  if (!s) return 9999;
  const t = s.trim().toLowerCase();
  if (t.startsWith('just') || t === 'now') return 0;
  const m = t.match(/(\d+)\s*([mhd])\s*ago/);
  if (!m) return 9999;
  const n = parseInt(m[1], 10);
  return m[2] === 'm' ? n : m[2] === 'h' ? n * 60 : n * 60 * 24;
}
```

Then find the existing Audit body in `renderInstanceDetail` (around line 4085):

```
  } else if (active === 'Audit') {
    body = `<div class="panel-block">
      ${panelHeading('Audit timeline', null)}
      <div class="timeline">
        <div class="tl-row"><div class="when">${escapeHtml(it.startedAt)}</div><div class="dot"></div><div class="lbl">Instance <strong>started</strong> on build ${escapeHtml(it.buildTag)}</div></div>
        ${it.status === 'failed' ? `<div class="tl-row bad"><div class="when">recently</div><div class="dot"></div><div class="lbl"><strong>Execution failed</strong> at ${escapeHtml(it.atStep)}</div></div>` : ''}
        ${it.status === 'completed' ? `<div class="tl-row"><div class="when">earlier</div><div class="dot"></div><div class="lbl">Instance <strong>completed</strong></div></div>` : ''}
      </div>
    </div>`;
  }
```

Replace with:

```
  } else if (active === 'Audit') {
    body = renderAuditTimeline(it);
  }
```

Then add `renderAuditTimeline` immediately after `renderActivityTree`:

```js
function renderAuditTimeline(it) {
  // Build a unified set of timeline events: lifecycle + notifications, sorted by atSortable (minutes ago, ascending = most recent first)
  const events = [];
  // Lifecycle: started
  events.push({
    atSortable: parseAgoMinutes(it.startedAt),
    at: it.startedAt,
    kind: 'lifecycle',
    label: `Instance <strong>started</strong> on build ${escapeHtml(it.buildTag)}`,
    bad: false,
  });
  if (it.status === 'failed') {
    events.push({
      atSortable: 1,
      at: 'recently',
      kind: 'lifecycle',
      label: `<strong>Execution failed</strong> at ${escapeHtml(it.atStep)}`,
      bad: true,
    });
  }
  if (it.status === 'completed') {
    events.push({
      atSortable: Math.max(0, parseAgoMinutes(it.startedAt) - 1),
      at: 'earlier',
      kind: 'lifecycle',
      label: 'Instance <strong>completed</strong>',
      bad: false,
    });
  }
  // Notifications
  (it.notifications || []).forEach(n => {
    const channelLabel = { 'email': 'Email', 'webhook': 'Webhook', 'in-app': 'In-app' }[n.channel] || n.channel;
    const verb = n.status === 'failed' ? 'failed' : 'sent';
    events.push({
      atSortable: typeof n.atSortable === 'number' ? n.atSortable : parseAgoMinutes(n.at),
      at: n.at,
      kind: 'notif',
      label: `<strong>${escapeHtml(channelLabel)}</strong> ${escapeHtml(verb)} → <code style="font-family:'JetBrains Mono',monospace;font-size:11px;">${escapeHtml(n.target)}</code><div style="color:var(--text-muted);margin-top:2px;">${escapeHtml(n.subject)}</div>`,
      bad: n.status === 'failed',
    });
  });
  // Sort: most recent first (smallest atSortable first)
  events.sort((a, b) => a.atSortable - b.atSortable);

  return `${docBanner('Audit timeline.', 'Lifecycle events (started, paused, completed) plus outgoing notifications (email, webhook, in-app) in one chronological log.', `inst-audit-${it.id}`)}
    <div class="panel-block">
      ${panelHeading('Audit timeline', null)}
      <div class="timeline">
        ${events.map(e => {
          const rowClass = e.bad ? 'tl-row bad' : 'tl-row';
          const dot = e.kind === 'notif'
            ? `<div class="dot notif">${e.bad ? '!' : '✉'}</div>`
            : `<div class="dot"></div>`;
          return `<div class="${rowClass}"><div class="when">${escapeHtml(e.at)}</div>${dot}<div class="lbl">${e.label}</div></div>`;
        }).join('')}
      </div>
    </div>`;
}
```

- [ ] **Step 3: Add the notification-dot CSS variant**

Find the existing timeline CSS (line 1262 onward):

```
.timeline .tl-row .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--border-strong); margin-top: 6px; flex-shrink: 0; }
.timeline .tl-row.bad .dot { background: var(--red-500); }
.timeline .tl-row.deploy .dot { background: var(--purple-500); }
```

Replace with:

```
.timeline .tl-row .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--border-strong); margin-top: 6px; flex-shrink: 0; }
.timeline .tl-row.bad .dot { background: var(--red-500); }
.timeline .tl-row.deploy .dot { background: var(--purple-500); }
.timeline .tl-row .dot.notif { width: 14px; height: 14px; border-radius: 4px; background: var(--blue-50); color: var(--blue-700); margin-top: 2px; font-size: 9px; line-height: 14px; text-align: center; font-weight: 700; border: 1px solid var(--blue-100); }
.timeline .tl-row.bad .dot.notif { background: var(--red-50); color: var(--red-700); border-color: var(--red-100); }
```

- [ ] **Step 4: Verify in browser**

- Open `inst_19f3d4` KYC review → Audit → expect 4 rows: started, "Execution failed", and two notifications interleaved (email failed, in-app sent). Failed email shows a red square with `!`; in-app shows a blue square with `✉`. Subject lines appear under each notification.
- Open `inst_55a1c0` → Audit → expect "started" + the in-app loan task notification.
- Open `inst_d5e6f7` Welcome email → Audit → expect started + Execution failed + the failed email notification.
- Open any instance with no `notifications` (e.g. `inst_b73e1c`) → Audit → only lifecycle rows; no breakage.

- [ ] **Step 5: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): weave outgoing notifications into Instance audit timeline (fix-list 5c)"
```

---

## Task 4: Token expand — data + CSS + STATE + renderer

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.instances entries; CSS section; STATE init; `renderActivityTree` Active tokens section)

**Closes:** Fix-list 5d

- [ ] **Step 1: Extend STATE with the expanded-tokens Set**

Find the existing STATE block (line 2120):

```
const STATE = {
  currentRoute: null,
  detail: { open: false, kind: null, id: null, tab: 0 },
  glossary: { open: false, scrollToTerm: null },
```

Replace with:

```
const STATE = {
  currentRoute: null,
  detail: { open: false, kind: null, id: null, tab: 0 },
  expandedTokens: new Set(),
  glossary: { open: false, scrollToTerm: null },
```

Then find `STATE.detail = { open: false, ... };` inside `closeDetail()` (around line 3959) and add a reset for `expandedTokens` immediately before that line. Target:

```js
function closeDetail() {
  STATE.detail = { open: false, kind: null, id: null, tab: 0 };
```

Replace with:

```js
function closeDetail() {
  STATE.expandedTokens.clear();
  STATE.detail = { open: false, kind: null, id: null, tab: 0 };
```

- [ ] **Step 2: Add a `toggleToken` global helper near `dismissBanner`**

Find `function dismissBanner(id)` (around line 4184) and add immediately after its closing brace:

```js
function toggleToken(tokenId) {
  if (STATE.expandedTokens.has(tokenId)) STATE.expandedTokens.delete(tokenId);
  else STATE.expandedTokens.add(tokenId);
  renderDetail();
}
```

- [ ] **Step 3: Add `tokens` arrays to mock process instances**

Apply via `Edit` (matching the unique opening of each existing object literal). Tokens to add per instance:

`inst_19f3d4` KYC review (failed) — add:

```js
tokens: [
  { id: 'tok_p3', name: 'Token tok_p3', state: 'FINISHED_WITH_ERROR', stateDesc: 'raised exception during execution', currentNode: 'Document upload',
    nodesActionStates: [
      { at: '3m ago', node: 'Document upload', action: 'execute', status: 'error: NullPointerException at DocumentService.scan()' },
      { at: '5m ago', node: 'Document upload', action: 'enter', status: 'token arrived' },
      { at: '6m ago', node: 'Consent gate', action: 'complete', status: 'ok' },
      { at: '7m ago', node: 'Start', action: 'complete', status: 'ok' },
    ] },
],
```

`inst_55a1c0` Loan application (waiting) — add:

```js
tokens: [
  { id: 'tok_loan_main', name: 'Token tok_loan_main', state: 'ON_HOLD', stateDesc: 'paused awaiting input', currentNode: 'Manager approval',
    nodesActionStates: [
      { at: '12m ago', node: 'Manager approval', action: 'enter', status: 'waiting on task task_1' },
      { at: '13m ago', node: 'Risk score', action: 'complete', status: 'risk=0.42' },
      { at: '14m ago', node: 'Application intake', action: 'complete', status: 'ok' },
    ],
    backSeq: { node: 'Application intake', reason: 'returned for missing field — corrected and resumed' } },
],
```

`inst_a01b3e` Nightly retention sweep (terminated) — add:

```js
tokens: [
  { id: 'tok_sweep_a', name: 'Token tok_sweep_a', state: 'ABORTED', stateDesc: 'manually terminated', currentNode: 'Process users',
    nodesActionStates: [
      { at: '2h ago', node: 'Process users', action: 'abort', status: 'aborted by ops' },
      { at: '2h ago', node: 'Process users', action: 'enter', status: 'token arrived' },
    ] },
  { id: 'tok_sweep_b', name: 'Token tok_sweep_b', state: 'ABORTED', stateDesc: 'manually terminated', currentNode: 'Process logs',
    nodesActionStates: [
      { at: '2h ago', node: 'Process logs', action: 'abort', status: 'aborted by ops' },
      { at: '2h ago', node: 'Process logs', action: 'enter', status: 'token arrived' },
    ] },
],
```

`inst_7c20fa` Document classifier (running) — add:

```js
tokens: [
  { id: 'tok_cls_main', name: 'Token tok_cls_main', state: 'STARTED', stateDesc: 'executing', currentNode: 'AI scoring',
    nodesActionStates: [
      { at: '5m ago', node: 'AI scoring', action: 'enter', status: 'running' },
      { at: '10m ago', node: 'OCR', action: 'complete', status: 'ok · 14 pages' },
      { at: '31m ago', node: 'Receive batch', action: 'complete', status: 'ok' },
    ] },
  { id: 'tok_cls_aux', name: 'Token tok_cls_aux', state: 'STARTED', stateDesc: 'executing', currentNode: 'Verifier',
    nodesActionStates: [
      { at: '4m ago', node: 'Verifier', action: 'enter', status: 'running' },
      { at: '11m ago', node: 'Classifier', action: 'complete', status: 'class=invoice' },
    ] },
],
```

Use `Edit` to insert these after the `variables: { ... }` line (or after `notifications` / `initiator` if present) and before the final `}` of each instance object.

- [ ] **Step 4: Update the "Active tokens" section of `renderActivityTree` to render and expand**

Find the existing process branch:

```js
  // Process: tokens + subprocesses
  return `${docBanner("What's an Activity tree?", "For processes, this shows active tokens (where execution pointers are right now) and any spawned sub-processes.", 'inst-activity-tree-p')}
    <div class="panel-block">
      ${panelHeading('Active tokens', 'Token')}
      <div class="tree-node ${it.status === 'failed' ? 'bad' : ''}">
        <span style="color:var(--green-500)">▸</span>
        <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
        <span class="pos">${it.status}${it.activeIncidentCount ? ` · ${it.activeIncidentCount} incidents` : ''}</span>
      </div>
    </div>
    <div class="panel-block">
      ${panelHeading('Subprocesses', null)}
      ${subprocessesHtml || '<div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">This instance hasn\'t spawned any subprocesses.</div>'}
    </div>`;
}
```

Replace with:

```js
  // Process: tokens + subprocesses
  const tokensHtml = (it.tokens && it.tokens.length)
    ? it.tokens.map(t => renderTokenRow(t, it)).join('')
    : `<div class="tree-node ${it.status === 'failed' ? 'bad' : ''}">
        <span style="color:var(--green-500)">▸</span>
        <span class="lbl"><strong>${escapeHtml(it.definitionName)}</strong> · ${escapeHtml(it.atStep)}</span>
        <span class="pos">${it.status}${it.activeIncidentCount ? ` · ${it.activeIncidentCount} incidents` : ''}</span>
      </div>`;
  return `${docBanner("What's an Activity tree?", "For processes, this shows active tokens (where execution pointers are right now) and any spawned sub-processes.", 'inst-activity-tree-p')}
    <div class="panel-block">
      ${panelHeading('Active tokens', 'Token')}
      ${tokensHtml}
    </div>
    <div class="panel-block">
      ${panelHeading('Subprocesses', null)}
      ${subprocessesHtml || '<div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">This instance hasn\'t spawned any subprocesses.</div>'}
    </div>`;
}

function renderTokenRow(t, it) {
  const expanded = STATE.expandedTokens.has(t.id);
  const isError = /ERROR|FAILED|ABORTED/i.test(t.state);
  const chevron = expanded ? '▼' : '▶';
  const rowClass = isError ? 'tree-node bad token-row' : 'tree-node token-row';
  const head = `<div class="${rowClass}" onclick="toggleToken('${t.id}')" style="cursor:pointer;">
    <span style="color:var(--text-dim);font-size:10px;width:10px;">${chevron}</span>
    <span style="color:var(--green-500)">▸</span>
    <span class="lbl"><strong>${escapeHtml(t.name)}</strong> · ${escapeHtml(t.currentNode)}</span>
    <span class="pos">${escapeHtml(t.state)}</span>
  </div>`;
  if (!expanded) return head;
  const history = (t.nodesActionStates || []).map(h => `
    <div class="token-history-row">
      <span class="when">${escapeHtml(h.at)}</span>
      <span class="node">${escapeHtml(h.node)}</span>
      <span class="action">${escapeHtml(h.action)}</span>
      <span class="status">${escapeHtml(h.status)}</span>
    </div>`).join('');
  const back = t.backSeq
    ? `<div class="token-back">↺ Back to <strong>${escapeHtml(t.backSeq.node)}</strong> — <span style="color:var(--text-muted);">${escapeHtml(t.backSeq.reason)}</span></div>`
    : '';
  const body = `<div class="token-expand">
    <div class="token-meta">
      <span class="raw-state ${isError ? 'bad' : ''}" data-tip="${escapeHtml(t.stateDesc || '')}">${escapeHtml(t.state)}</span>
      <span class="raw-state-desc">${escapeHtml(t.stateDesc || '')}</span>
    </div>
    ${back}
    <div class="token-history">
      <div class="token-history-row head"><span class="when">When</span><span class="node">Node</span><span class="action">Action</span><span class="status">Status</span></div>
      ${history || '<div style="font-size:11px;color:var(--text-dim);padding:6px;">No node action history recorded.</div>'}
    </div>
  </div>`;
  return head + body;
}
```

- [ ] **Step 5: Add CSS for token rows**

Find the existing tree CSS block (line 1096 onward) and after the `.tree-child::before` rule, append:

```css
/* Token row + expand panel */
.tree-node.token-row { background: white; }
.tree-node.token-row:hover { background: var(--bg-alt); }
.token-expand {
  margin: -2px 0 10px 0;
  padding: 10px 12px;
  background: var(--bg-soft, var(--bg-alt));
  border: 1px solid var(--border);
  border-radius: 0 0 6px 6px;
  font-size: 12px;
}
.token-expand .token-meta {
  display: flex; align-items: center; gap: 10px;
  margin-bottom: 8px;
}
.token-expand .raw-state {
  font-family: 'JetBrains Mono', monospace; font-size: 11px;
  padding: 2px 8px; border-radius: 9999px;
  background: var(--green-50, #e6f2ef); color: var(--green-700, #005b44);
  border: 1px solid var(--green-100, #b0d8ce);
}
.token-expand .raw-state.bad { background: var(--red-50); color: var(--red-700); border-color: var(--red-100); }
.token-expand .raw-state-desc { color: var(--text-muted); font-size: 11px; }
.token-expand .token-back {
  display: inline-block;
  padding: 4px 10px;
  margin-bottom: 8px;
  background: var(--yellow-50);
  border: 1px solid var(--yellow-100, #f3e0a7);
  border-radius: 6px;
  color: var(--yellow-700);
  font-size: 11px;
}
.token-expand .token-history {
  display: flex; flex-direction: column;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: white;
  overflow: hidden;
}
.token-expand .token-history-row {
  display: grid;
  grid-template-columns: 80px 130px 80px 1fr;
  gap: 8px;
  padding: 6px 10px;
  font-size: 11px;
  border-bottom: 1px solid var(--border);
}
.token-expand .token-history-row:last-child { border-bottom: none; }
.token-expand .token-history-row .when { color: var(--text-dim); font-variant-numeric: tabular-nums; }
.token-expand .token-history-row .node { color: var(--text); font-weight: 600; }
.token-expand .token-history-row .action { color: var(--purple-700); font-family: 'JetBrains Mono', monospace; }
.token-expand .token-history-row .status { color: var(--text-muted); }
.token-expand .token-history-row.head { background: var(--bg-alt); font-size: 10px; text-transform: uppercase; letter-spacing: 0.04em; color: var(--text-dim); font-weight: 700; }
.token-expand .token-history-row.head .node,
.token-expand .token-history-row.head .action,
.token-expand .token-history-row.head .status,
.token-expand .token-history-row.head .when { color: var(--text-dim); font-weight: 700; }
```

Note: if `--green-50`, `--green-700`, `--green-100`, `--yellow-100`, `--bg-soft`, or `--red-100` aren't defined in the prototype's `:root`, the `var(...)` fallbacks above keep the visuals legible. Confirm by grepping `:root` in the file; if any is genuinely missing, leave the CSS as written — the fallback hex values handle it.

- [ ] **Step 6: Verify in browser**

- Open `inst_19f3d4` → Activity tree → Active tokens shows one row `Token tok_p3` with a chevron and red background. Click → expands. Expanded body shows pill `FINISHED_WITH_ERROR` (red), description "raised exception during execution", and 4 rows of node action history.
- Open `inst_55a1c0` → Activity tree → click token → expanded body shows `ON_HOLD` pill, a "↺ Back to Application intake — returned for missing field..." chip, and 3-row history.
- Open `inst_a01b3e` → expect two tokens, both showing `ABORTED` pill when expanded.
- Open `inst_91ab23` (completed, no `tokens`) → Active tokens still renders the legacy single-line fallback. Activity tree section stays usable.
- Close detail panel and reopen another instance — `STATE.expandedTokens` resets cleanly (no leakage).

- [ ] **Step 7: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): expandable token detail with raw state + node history (fix-list 5d)"
```

---

## Task 5: Variables edit — CSS + STATE + modal + renderer

**Files:**
- Modify: `runtime-prototype.html` (STATE init; CSS; new modal HTML + JS; `renderInstanceDetail` Variables body)

**Closes:** Fix-list 5e

- [ ] **Step 1: Extend STATE with `variableEdits`**

Find the STATE block we already edited in Task 4:

```js
const STATE = {
  currentRoute: null,
  detail: { open: false, kind: null, id: null, tab: 0 },
  expandedTokens: new Set(),
  glossary: { open: false, scrollToTerm: null },
```

Replace with:

```js
const STATE = {
  currentRoute: null,
  detail: { open: false, kind: null, id: null, tab: 0 },
  expandedTokens: new Set(),
  variableEdits: {},          // { [instanceId]: { [key]: newValue } }
  variableEditModal: { open: false, instanceId: null, key: null, originalValue: null },
  glossary: { open: false, scrollToTerm: null },
```

- [ ] **Step 2: Add the modal HTML scaffold**

Find the existing detail overlay block (around line 1611):

```
<div class="detail-overlay" id="detailOverlay"></div>
```

Append immediately after it:

```
<div class="app-modal-scrim" id="appModalScrim" onclick="closeVariableEditModal()"></div>
<div class="app-modal" id="appModal" role="dialog" aria-modal="true">
  <div class="app-modal-head">
    <h3 id="appModalTitle">Edit variable</h3>
    <button class="x" onclick="closeVariableEditModal()" aria-label="Close">×</button>
  </div>
  <div class="app-modal-body" id="appModalBody"></div>
  <div class="app-modal-foot">
    <a id="appModalRevert" style="color:var(--blue-500);cursor:pointer;font-size:12px;font-weight:600;display:none;" onclick="revertVariableEdit()">Revert to original</a>
    <div style="flex:1"></div>
    <button class="btn ghost" onclick="closeVariableEditModal()">Cancel</button>
    <button class="btn primary" onclick="saveVariableEdit()">Save</button>
  </div>
</div>
```

- [ ] **Step 3: Add the modal CSS**

Find the existing `.detail-overlay` CSS (around line 696). After the `.detail-overlay.open` rule, append:

```css
/* Generic centered modal scaffold (.app-modal) */
.app-modal-scrim {
  position: fixed; inset: 0;
  background: rgba(20, 30, 50, 0.45);
  backdrop-filter: blur(2px);
  opacity: 0; pointer-events: none;
  transition: opacity 0.15s ease;
  z-index: 60;
}
.app-modal-scrim.open { opacity: 1; pointer-events: auto; }
.app-modal {
  position: fixed; top: 50%; left: 50%;
  transform: translate(-50%, calc(-50% + 8px));
  width: 460px; max-width: calc(100vw - 32px);
  background: white;
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: 0 24px 56px rgba(20, 30, 50, 0.25);
  opacity: 0; pointer-events: none;
  transition: opacity 0.15s ease, transform 0.15s ease;
  z-index: 61;
  display: flex; flex-direction: column;
}
.app-modal.open {
  opacity: 1; pointer-events: auto;
  transform: translate(-50%, -50%);
}
.app-modal-head { display: flex; justify-content: space-between; align-items: center; padding: 14px 18px; border-bottom: 1px solid var(--border); }
.app-modal-head h3 { font-size: 14px; font-weight: 700; margin: 0; }
.app-modal-head .x { background: none; border: 0; cursor: pointer; font-size: 18px; color: var(--text-dim); padding: 0; line-height: 1; }
.app-modal-head .x:hover { color: var(--text); }
.app-modal-body { padding: 16px 18px; font-size: 13px; }
.app-modal-foot { display: flex; gap: 8px; align-items: center; padding: 12px 18px; border-top: 1px solid var(--border); background: var(--bg-alt); border-radius: 0 0 12px 12px; }
.app-modal-body .field-label { font-size: 11px; text-transform: uppercase; letter-spacing: 0.04em; color: var(--text-dim); font-weight: 700; margin-bottom: 4px; }
.app-modal-body .field-readonly {
  font-family: 'JetBrains Mono', monospace; font-size: 12px;
  padding: 6px 10px; background: var(--bg-alt); border: 1px solid var(--border);
  border-radius: 6px; color: var(--text); margin-bottom: 12px;
}
.app-modal-body textarea {
  width: 100%; min-height: 70px;
  font-family: 'JetBrains Mono', monospace; font-size: 12px;
  padding: 8px 10px; border: 1px solid var(--border-strong);
  border-radius: 6px; resize: vertical; box-sizing: border-box;
  color: var(--text); background: white;
}
.app-modal-body textarea:focus { outline: 2px solid var(--blue-500); outline-offset: -1px; border-color: var(--blue-500); }

/* Variables tab pencil + edited badge */
.var-row { position: relative; }
.var-row:hover .var-edit-btn { opacity: 1; }
.var-edit-btn {
  display: inline-flex; align-items: center; gap: 4px;
  background: none; border: 0; cursor: pointer;
  color: var(--blue-500); font-size: 11px; font-weight: 600;
  padding: 2px 6px; border-radius: 4px;
  opacity: 0; transition: opacity 0.1s ease;
  margin-left: 8px;
}
.var-edit-btn:hover { background: var(--blue-50); }
.var-edited-badge {
  display: inline-block;
  margin-left: 8px;
  padding: 1px 6px;
  font-size: 9px; font-weight: 700;
  text-transform: uppercase; letter-spacing: 0.04em;
  background: var(--yellow-50); color: var(--yellow-700);
  border: 1px solid var(--yellow-100, #f3e0a7);
  border-radius: 9999px;
}
```

- [ ] **Step 4: Add modal open/close/save/revert functions**

Add immediately after `dismissBanner` and `toggleToken` (which were added in earlier tasks):

```js
function openVariableEditModal(instanceId, key) {
  const it = MOCK_DATA.instances.find(x => x.id === instanceId);
  if (!it) return;
  const original = it.variables[key];
  const edited = STATE.variableEdits[instanceId]?.[key];
  const current = edited !== undefined ? edited : original;
  STATE.variableEditModal = { open: true, instanceId, key, originalValue: original };
  document.getElementById('appModalTitle').textContent = `Edit variable · ${key}`;
  document.getElementById('appModalBody').innerHTML = `
    <div class="field-label">Key</div>
    <div class="field-readonly">${escapeHtml(key)}</div>
    <div class="field-label">Original value</div>
    <div class="field-readonly">${escapeHtml(String(original))}</div>
    <div class="field-label">New value</div>
    <textarea id="appModalInput">${escapeHtml(String(current))}</textarea>
    <div style="font-size:11px;color:var(--text-dim);margin-top:8px;">In-session change · requires <code style="font-family:'JetBrains Mono',monospace;font-size:10px;">wks_process_instance_variables_edit</code> · not persisted.</div>
  `;
  document.getElementById('appModalRevert').style.display = edited !== undefined ? 'inline' : 'none';
  document.getElementById('appModalScrim').classList.add('open');
  document.getElementById('appModal').classList.add('open');
  setTimeout(() => document.getElementById('appModalInput').focus(), 50);
}
function closeVariableEditModal() {
  STATE.variableEditModal = { open: false, instanceId: null, key: null, originalValue: null };
  document.getElementById('appModalScrim').classList.remove('open');
  document.getElementById('appModal').classList.remove('open');
}
function saveVariableEdit() {
  const { instanceId, key, originalValue } = STATE.variableEditModal;
  if (!instanceId) return;
  const raw = document.getElementById('appModalInput').value;
  // Try to coerce back to number if the original was numeric and the input parses cleanly
  let newValue = raw;
  if (typeof originalValue === 'number' && raw.trim() !== '' && !isNaN(Number(raw))) {
    newValue = Number(raw);
  }
  if (!STATE.variableEdits[instanceId]) STATE.variableEdits[instanceId] = {};
  STATE.variableEdits[instanceId][key] = newValue;
  closeVariableEditModal();
  renderDetail();
}
function revertVariableEdit() {
  const { instanceId, key } = STATE.variableEditModal;
  if (!instanceId || !STATE.variableEdits[instanceId]) return closeVariableEditModal();
  delete STATE.variableEdits[instanceId][key];
  if (Object.keys(STATE.variableEdits[instanceId]).length === 0) delete STATE.variableEdits[instanceId];
  closeVariableEditModal();
  renderDetail();
}
```

- [ ] **Step 5: Hook Esc + scrim into the existing key handler**

Find the existing `else if (STATE.detail.open) closeDetail();` (around line 4749). Replace the surrounding key handler with the same block plus a modal-close branch. Target:

```js
    else if (STATE.detail.open) closeDetail();
```

Replace with:

```js
    else if (STATE.variableEditModal.open) closeVariableEditModal();
    else if (STATE.detail.open) closeDetail();
```

- [ ] **Step 6: Update the Variables tab renderer**

Find the existing Variables body (line 4061 onward):

```js
  } else if (active === 'Variables') {
    body = `<div class="panel-block">
      ${panelHeading('Variables', null)}
      <table style="width:100%;font-size:12px;">
        ${Object.entries(it.variables || {}).map(([k, v]) => `<tr>
          <td style="padding:6px 0;color:var(--text-muted);width:140px;font-family:'JetBrains Mono',monospace;font-size:11px;">${escapeHtml(k)}</td>
          <td style="padding:6px 0;color:var(--text);font-family:'JetBrains Mono',monospace;font-size:11px;">${escapeHtml(String(v))}</td>
        </tr>`).join('')}
      </table>
    </div>`;
  }
```

Replace with:

```js
  } else if (active === 'Variables') {
    body = renderVariablesTab(it);
  }
```

Then add `renderVariablesTab` immediately after `renderInstanceDetail`:

```js
function renderVariablesTab(it) {
  const edits = STATE.variableEdits[it.id] || {};
  const entries = Object.entries(it.variables || {});
  const rows = entries.map(([k, v]) => {
    const edited = edits[k] !== undefined;
    const display = edited ? edits[k] : v;
    const badge = edited ? '<span class="var-edited-badge">edited</span>' : '';
    return `<tr class="var-row">
      <td style="padding:6px 0;color:var(--text-muted);width:160px;font-family:'JetBrains Mono',monospace;font-size:11px;vertical-align:top;">${escapeHtml(k)}</td>
      <td style="padding:6px 0;color:var(--text);font-family:'JetBrains Mono',monospace;font-size:11px;vertical-align:top;">
        ${escapeHtml(String(display))}${badge}
        <button class="var-edit-btn" onclick="openVariableEditModal('${it.id}','${escapeHtml(k)}')" title="Edit variable">✎ Edit</button>
      </td>
    </tr>`;
  }).join('');
  return `${docBanner('Edit variables.', 'Inline edits go to a live running instance. Requires the wks_process_instance_variables_edit permission (always granted in this prototype).', 'inst-variables-edit')}
    <div class="panel-block">
      ${panelHeading('Variables', null)}
      <table style="width:100%;font-size:12px;">${rows}</table>
    </div>`;
}
```

- [ ] **Step 7: Verify in browser**

- Open `inst_55a1c0` Loan application → Variables. Doc banner appears. Hover the `amount` row — a pencil "✎ Edit" appears at the right. Click it.
- Modal appears with key `amount`, original value `42000`, current value `42000` in a textarea.
- Change to `99999` → Save → modal closes, row now shows `99999` with an `edited` badge.
- Click "✎ Edit" again — modal shows the "Revert to original" link at bottom-left.
- Click "Revert to original" → row reverts to `42000`, badge disappears.
- Press Esc with modal open → modal closes.
- Click the scrim with modal open → modal closes.
- Open `inst_b73e1c` Payment retry → Variables → confirm the same UX works (no breakage on different value types).

- [ ] **Step 8: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): variable edit affordance with in-session modal (fix-list 5e)"
```

---

## Task 6: Doc-banner copy + fix-list close

**Files:**
- Modify: `runtime-prototype.html` (touch-up only — banner copy already added in tasks 1–5)
- Modify: `fix-list.md`

- [ ] **Step 1: Confirm banner ids unique and dismissible**

Grep `docBanner(` in the file to confirm every new banner uses a distinct id. Expected banner ids introduced in this plan:

- `inst-activity-tree` (existing)
- `inst-activity-tree-p` (existing)
- `inst-audit-<id>` — generated per-instance via template literal (Task 3)
- `inst-variables-edit` (Task 5)

If `inst-audit-<id>` proves noisy across instances, simplify to a fixed `inst-audit` id and remove the template substitution. Use `Edit` on Task 3's renderer:

Old:

```js
return `${docBanner('Audit timeline.', 'Lifecycle events (started, paused, completed) plus outgoing notifications (email, webhook, in-app) in one chronological log.', `inst-audit-${it.id}`)}
```

New (if simplifying):

```js
return `${docBanner('Audit timeline.', 'Lifecycle events (started, paused, completed) plus outgoing notifications (email, webhook, in-app) in one chronological log.', 'inst-audit')}
```

Run the acceptance walkthrough one more time to make sure dismissal sticks across instances.

- [ ] **Step 2: Walk acceptance criteria from the spec**

For each:

- [x] Opening `sess_a82c91` shows two subprocesses in Activity tree; clicking each navigates
- [x] Opening `inst_19f3d4` Overview shows `Trigger · UI Flow · sess_a82c91`
- [x] Opening `inst_19f3d4` Audit shows the failed email notification interleaved chronologically
- [x] Opening `inst_19f3d4` Activity tree → clicking the token reveals raw state + history
- [x] Opening `inst_55a1c0` Variables → edit `amount` → Save → badge → Revert works
- [x] Doc banner dismissal persists across panel re-opens (sessionStorage-backed via existing pattern)

Any item that fails: drop back to the appropriate task and patch.

- [ ] **Step 3: Update `fix-list.md` to mark 5a–5e DONE**

Find the existing Tier 2 section for item 5:

```
### 5. Process Instance detail tabs are incomplete
**Spec:** §5 — eight tabs: Variables, Tokens, Subprocesses, Exceptions, Workflows, Trigger, Notifications, Audit.
**Prototype state:** four to six tabs: Overview, Variables, Activity tree, AI runs, [Incidents if any], Audit.
**Missing pieces:**
- **5a.** Subprocesses — Activity tree shows "No subprocesses for this instance" (line 3134); mock data has no sub-process tree (only `spawnedBy` in the other direction).
- **5b.** Trigger tab — initiator info not surfaced.
- **5c.** Notifications tab — outgoing notifications surface absent.
- **5d.** Token detail — Activity tree mentions tokens but doesn't surface `state` (ACTIVE/ON_HOLD/etc.), `nodesActionStates[]` per-node history, or `backSeq` navigation.
- **5e.** Variables edit affordance gated on `wks_process_instance_variables_edit` — variables are read-only.
```

Replace with:

```
### 5. Process Instance detail tabs are incomplete  ✅ **DONE**
**Spec:** §5 — eight tabs: Variables, Tokens, Subprocesses, Exceptions, Workflows, Trigger, Notifications, Audit.
**Resolution:** the redesign collapsed the 8 starter tabs into 6 (Overview · Variables · Activity tree · AI runs · Incidents · Audit). Closing the remaining gaps without expanding the IA:
- **5a.** Subprocesses ✅ — `subprocesses[]` added to representative mock instances + new `inst_doc_scan` grandchild; `renderActivityTree` now renders a recursive subprocess tree under the existing "Subprocesses" section. Rows are click-through to child instance detail.
- **5b.** Trigger initiator ✅ — `initiator: { kind, triggerId?, triggerName?, parentId?, userId?, when? }` added to mock instances. Overview gains a `Trigger` KV row with kind icon (manual/cron/webhook/process/uiflow) and a linkable trigger or parent instance reference. Trigger tab not reintroduced — folded into Overview to preserve the 6-tab IA.
- **5c.** Notifications ✅ — `notifications[]` (channel, target, subject, status) added. `renderAuditTimeline` interleaves lifecycle events + notifications sorted by `atSortable` (minutes ago), with envelope dot for sent and red `!` dot for failed. Audit banner updated to call out notifications. Notifications tab not reintroduced — folded into Audit.
- **5d.** Token detail ✅ — `tokens[]` added (id, name, state, stateDesc, currentNode, nodesActionStates[], backSeq?). Each token in Active tokens is an expandable row; expansion reveals raw state pill (with desc), back-seq chip when present, and a node-by-node action history table. `STATE.expandedTokens` resets on detail close.
- **5e.** Variables edit ✅ — `STATE.variableEdits` overlay + per-row pencil affordance + new `.app-modal` scaffold (centered, Esc + scrim close). Save persists in-session, edited values display with an `edited` badge, modal exposes a "Revert to original" link when the row carries an edit. Permission is hard-coded as granted with a doc banner that names `wks_process_instance_variables_edit`.

The 6-tab IA survives intact; the new `.app-modal` scaffold is available for any future modal needs.
```

- [ ] **Step 4: Final commit**

```bash
git add runtime-prototype.html fix-list.md
git commit -m "docs(runtime): close fix-list 5a-5e with banner polish + DONE markers"
```

---

## Self-review checklist (executed by plan author)

- **Spec coverage:** All five sub-items (5a–5e) have a dedicated task. Out-of-scope items in spec (backend persistence, modeling other permissions) remain explicitly excluded.
- **Placeholder scan:** No TBD / TODO / "implement later" / "appropriate error handling" / "similar to Task N" tokens. Every step has code.
- **Type consistency:** `STATE.expandedTokens` is `Set<string>` everywhere. `STATE.variableEdits` is `Record<instanceId, Record<key, newValue>>` everywhere. `renderSubprocessTree`, `renderTokenRow`, `renderAuditTimeline`, `renderVariablesTab`, `renderInitiator`, `parseAgoMinutes`, `toggleToken`, `openVariableEditModal`, `closeVariableEditModal`, `saveVariableEdit`, `revertVariableEdit` are named consistently across tasks.
- **Scope:** Single source file plus one markdown update. Each task ends in a working state and a commit. No unrelated refactoring.
