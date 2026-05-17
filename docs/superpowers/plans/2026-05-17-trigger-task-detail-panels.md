# Trigger + Task Detail Panels Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the literal "Detail content coming in Task 8" stubs for `renderTriggerDetail` and `renderTaskDetail` with full right-slide panels matching the rest of the prototype's chrome and closing fix-list Tier 2 #6.

**Architecture:** Two new render functions following the established `[head, tabs, body]` triple convention used by `renderInstanceDetail` / `renderIncidentDetail` / `renderWorkflowRunDetail`. Trigger panel has Configuration / Recent fires / Failures (conditional) tabs; Task panel has Overview / Form / History tabs. In-session mutation for Claim/Complete is done by direct mutation of `MOCK_DATA.tasks` entries — the existing `claimTask(id)` helper at line 4172 already follows this pattern, so adding `completeTask(id)` matches it (this is a small but conscious deviation from the spec, which proposed a `STATE.taskMutations` overlay; matching the existing helper is simpler and avoids parallel mutation paths).

**Tech Stack:** Single-file HTML/CSS/vanilla-JS prototype. No build step, no test framework. Verification is visual via a local browser open of `runtime-prototype.html`.

**Spec:** [`docs/superpowers/specs/2026-05-17-trigger-task-detail-panels-design.md`](../specs/2026-05-17-trigger-task-detail-panels-design.md)

---

## Conventions used in this plan

- Every code change is shown in full. File paths are absolute from the prototype root.
- Verification: open `runtime-prototype.html` in a browser (`open runtime-prototype.html` on macOS), then follow the linked action; the expected outcome is described per step.
- Commit boundaries at the end of each task. Commits stay small and scoped.

## Deviations from the spec

The spec proposed `STATE.taskMutations: Record<taskId, { assignee?, completed? }>` as an overlay. The existing `claimTask(id)` helper (line 4172) mutates `MOCK_DATA.tasks[i].assignee` directly. To avoid two parallel mutation paths for the same data, this plan extends the existing pattern — `completeTask(id)` mutates `MOCK_DATA.tasks[i].completed = true`. Both pages (`renderTasks` and the new task detail panel) read directly from `MOCK_DATA`. Form data is synthesized at render time rather than stored on each task; history events are derived from current state. UX identical to the spec; less mock-data verbosity.

---

## Task 1: Trigger detail panel — Configuration / Recent fires / Failures

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.triggers entries; `renderTriggerDetail` body; possibly add a `.fr-mini` or reuse `.kv` / `.timeline` for the fires table)

**Closes:** Half of fix-list Tier 2 #6 (Trigger detail).

- [ ] **Step 1: Add `spawnsDefinitionId` + `audit` to each trigger**

Find the `MOCK_DATA.triggers` block (around line 1907 — `triggers: [...]`). Replace it entirely. Target:

```
  triggers: [
    { id: 'cron_retention', name: 'Nightly retention sweep', type: 'cron', schedule: '0 2 * * *', spawns: 'retention_v2', lastFire: '14:02 today', status: 'active', errorCount24h: 0 },
    { id: 'cron_kpi', name: 'Hourly KPI rollup', type: 'cron', schedule: '0 * * * *', spawns: 'kpi_rollup', lastFire: '15:00 today', status: 'active', errorCount24h: 0 },
    { id: 'cron_reconcile', name: 'Quarterly reconciliation', type: 'cron', schedule: '0 0 1 */3 *', spawns: 'reconcile', lastFire: 'Apr 1 02:00', status: 'paused', errorCount24h: 0 },
    { id: 'cron_cleanup', name: 'Cleanup logs', type: 'cron', schedule: '30 3 * * *', spawns: 'cleanup', lastFire: '03:30 today', status: 'active', errorCount24h: 0 },
    { id: 'webhook_sf_sync', name: 'Salesforce sync', type: 'webhook', schedule: 'on event', spawns: 'sf_sync', lastFire: '22m ago', status: 'erroring', errorCount24h: 1 },
    { id: 'webhook_stripe', name: 'Stripe webhook', type: 'webhook', schedule: 'on event', spawns: 'stripe_handler', lastFire: '5m ago', status: 'active', errorCount24h: 0 },
  ],
```

Replace with:

```
  triggers: [
    { id: 'cron_retention', name: 'Nightly retention sweep', type: 'cron', schedule: '0 2 * * *', spawns: 'retention_v2', spawnsDefinitionId: 'def_retention_v2', lastFire: '14:02 today', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'alice', createdDate: '92d ago', modifiedBy: 'alice', modifiedDate: '92d ago' } },
    { id: 'cron_kpi', name: 'Hourly KPI rollup', type: 'cron', schedule: '0 * * * *', spawns: 'kpi_rollup', spawnsDefinitionId: 'def_kpi_rollup', lastFire: '15:00 today', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'bob', createdDate: '180d ago', modifiedBy: 'bob', modifiedDate: '14d ago' } },
    { id: 'cron_reconcile', name: 'Quarterly reconciliation', type: 'cron', schedule: '0 0 1 */3 *', spawns: 'reconcile', spawnsDefinitionId: 'def_reconcile', lastFire: 'Apr 1 02:00', status: 'paused', errorCount24h: 0,
      audit: { createdBy: 'carlos', createdDate: '270d ago', modifiedBy: 'alice', modifiedDate: '45d ago' } },
    { id: 'cron_cleanup', name: 'Cleanup logs', type: 'cron', schedule: '30 3 * * *', spawns: 'cleanup', spawnsDefinitionId: 'def_cleanup', lastFire: '03:30 today', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'alice', createdDate: '120d ago', modifiedBy: 'alice', modifiedDate: '60d ago' } },
    { id: 'webhook_sf_sync', name: 'Salesforce sync', type: 'webhook', schedule: 'on event', spawns: 'sf_sync', spawnsDefinitionId: 'def_sf_sync', lastFire: '22m ago', status: 'erroring', errorCount24h: 1,
      audit: { createdBy: 'bob', createdDate: '210d ago', modifiedBy: 'carlos', modifiedDate: '3d ago' } },
    { id: 'webhook_stripe', name: 'Stripe webhook', type: 'webhook', schedule: 'on event', spawns: 'stripe_handler', spawnsDefinitionId: 'def_stripe_handler', lastFire: '5m ago', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'alice', createdDate: '60d ago', modifiedBy: 'alice', modifiedDate: '60d ago' } },
  ],
```

- [ ] **Step 2: Add `webhook_loans_api` trigger**

The Instance Overview's Trigger row (from Tier 2 #5) links to `webhook_loans_api`, but that trigger doesn't exist in `MOCK_DATA.triggers` — only in `MOCK_DATA.triggerFires`. Add it so the cross-link resolves cleanly.

Inside the array literal you just replaced, immediately AFTER `webhook_stripe`:

```
    { id: 'webhook_stripe', name: 'Stripe webhook', type: 'webhook', schedule: 'on event', spawns: 'stripe_handler', spawnsDefinitionId: 'def_stripe_handler', lastFire: '5m ago', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'alice', createdDate: '60d ago', modifiedBy: 'alice', modifiedDate: '60d ago' } },
  ],
```

becomes:

```
    { id: 'webhook_stripe', name: 'Stripe webhook', type: 'webhook', schedule: 'on event', spawns: 'stripe_handler', spawnsDefinitionId: 'def_stripe_handler', lastFire: '5m ago', status: 'active', errorCount24h: 0,
      audit: { createdBy: 'alice', createdDate: '60d ago', modifiedBy: 'alice', modifiedDate: '60d ago' } },
    { id: 'webhook_loans_api', name: 'Loans API webhook', type: 'webhook', schedule: 'on event', spawns: 'loan_application', spawnsDefinitionId: 'def_loan_application', lastFire: '31m ago', status: 'erroring', errorCount24h: 3,
      audit: { createdBy: 'bob', createdDate: '40d ago', modifiedBy: 'bob', modifiedDate: '12m ago' } },
  ],
```

- [ ] **Step 3: Replace the `renderTriggerDetail` stub with a real renderer**

Find the existing stub (line ~5077):

```js
function renderTriggerDetail(it, tab) {
  return [`<div class="topline"><span class="crumb">Trigger</span><span style="font-family:'JetBrains Mono',monospace">${escapeHtml(it.id)}</span><button class="x" onclick="closeDetail()">×</button></div><h3>${escapeHtml(it.name)}</h3><div class="sub">${escapeHtml(it.type)} · ${escapeHtml(it.status)}</div>`, '', '<div class="empty"><p>Detail content coming in Task 8.</p></div>'];
}
```

Replace with:

```js
function renderTriggerDetail(it, tabIdx) {
  const fires = MOCK_DATA.triggerFires.filter(f => f.triggerId === it.id);
  const failures = fires.filter(f => f.outcome === 'failed');
  const baseTabs = ['Configuration', 'Recent fires'];
  if (failures.length > 0) baseTabs.push('Failures');
  const tabs = baseTabs;

  const kindIcons = { cron: '⏱', webhook: '↘', event: '✦' };
  const kindLabels = { cron: 'Cron', webhook: 'Webhook', event: 'Event' };
  const statusPill = (status) => {
    const map = { active: ['var(--green-50)', 'var(--green-700)'], paused: ['var(--bg-alt)', 'var(--text-muted)'], erroring: ['var(--red-50)', 'var(--red-700)'] };
    const [bg, fg] = map[status] || map.active;
    return `<span style="display:inline-block;padding:1px 8px;border-radius:9999px;background:${bg};color:${fg};font-size:11px;font-weight:600;">${escapeHtml(status)}</span>`;
  };

  const head = `
    <div class="topline">
      <span class="crumb">Trigger details</span>
      <span style="color:var(--border-strong)">/</span>
      <span style="font-family:'JetBrains Mono',monospace">${escapeHtml(it.id)}</span>
      <button class="x" onclick="closeDetail()">×</button>
    </div>
    <h3>${escapeHtml(it.name)}</h3>
    <div class="sub"><span aria-hidden="true" style="color:var(--text-dim);margin-right:4px;">${kindIcons[it.type] || '·'}</span>${escapeHtml(kindLabels[it.type] || it.type)} · ${statusPill(it.status)}${it.lastFire ? ` · <span style="color:var(--text-dim);font-size:11px;">last fire ${escapeHtml(it.lastFire)}</span>` : ''}</div>`;

  const tabsHtml = tabs.map((t, i) => {
    let count = '';
    if (t === 'Recent fires' && fires.length > 0) count = `<span class="count">${fires.length}</span>`;
    if (t === 'Failures' && failures.length > 0) count = `<span class="count alert">${failures.length}</span>`;
    return `<button class="tab ${i === tabIdx ? 'active' : ''}" onclick="STATE.detail.tab = ${i}; renderDetail();">${t}${count}</button>`;
  }).join('');

  const active = tabs[tabIdx];
  let body = '';
  if (active === 'Configuration') {
    const audit = it.audit || {};
    body = `${docBanner("What's a trigger?", "Something that starts a process — on a schedule (cron), an incoming event (webhook), or an internal signal. Spawned instances appear in Instances.", 'trigger-config')}
      <div class="panel-block">
        ${panelHeading('Configuration', 'Trigger')}
        <div class="kv">
          <span>Type</span><span><span aria-hidden="true" style="color:var(--text-dim);margin-right:4px;">${kindIcons[it.type] || '·'}</span>${escapeHtml(kindLabels[it.type] || it.type)}</span>
          <span>Schedule</span><span>${it.type === 'cron' ? `<code style="font-family:'JetBrains Mono',monospace;font-size:11px;">${escapeHtml(it.schedule)}</code>` : escapeHtml(it.schedule)}</span>
          <span>Spawns</span><span>${escapeHtml(it.spawns)} <span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--text-muted);">${escapeHtml(it.spawnsDefinitionId || '')}</span></span>
          <span>Last fire</span><span>${escapeHtml(it.lastFire || '—')}</span>
          <span>Status</span><span>${statusPill(it.status)}</span>
          <span>Errors (24h)</span><span style="${it.errorCount24h > 0 ? 'color:var(--red-700);font-weight:600;' : ''}">${it.errorCount24h}</span>
          <span>Created</span><span>${escapeHtml(audit.createdBy || '—')} · ${escapeHtml(audit.createdDate || '—')}</span>
          <span>Modified</span><span>${escapeHtml(audit.modifiedBy || '—')} · ${escapeHtml(audit.modifiedDate || '—')}</span>
        </div>
      </div>`;
  } else if (active === 'Recent fires') {
    body = renderTriggerFiresList(fires, 'all');
  } else if (active === 'Failures') {
    body = renderTriggerFiresList(failures, 'failed');
  }

  return [head, tabsHtml, body];
}

function renderTriggerFiresList(fires, mode) {
  const heading = mode === 'failed' ? 'Failed fires' : 'Recent fires';
  if (fires.length === 0) {
    return `<div class="panel-block">
      ${panelHeading(heading, null)}
      <div style="font-size:11px;color:var(--text-dim);padding:8px 10px;">${mode === 'failed' ? 'No failures recorded for this trigger.' : 'No fires recorded for this trigger yet.'}</div>
    </div>`;
  }
  const rows = fires.map(f => {
    const incident = mode === 'failed'
      ? MOCK_DATA.incidents.find(i => i.sourceEntity && i.sourceEntity.kind === 'trigger' && i.sourceEntity.id === f.triggerId)
      : null;
    const tail = f.outcome === 'success'
      ? (f.spawnedInstanceId ? `<a style="color:var(--blue-500);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11px;" onclick="openDetail('instance','${escapeHtml(f.spawnedInstanceId)}')">${escapeHtml(f.spawnedInstanceId)}</a>` : '<span style="color:var(--text-muted)">—</span>')
      : `<span style="color:var(--red-700);">${escapeHtml(f.failureReason || 'failed')}</span>${incident ? ` · <a style="color:var(--blue-500);cursor:pointer;font-size:11px;font-weight:600;" onclick="openDetail('incident','${escapeHtml(incident.id)}')">Open incident →</a>` : ''}`;
    const outcomeLabel = f.outcome === 'success'
      ? '<span style="color:var(--green-700);font-weight:600;font-size:11px;">Success</span>'
      : '<span style="color:var(--red-700);font-weight:600;font-size:11px;">Failed</span>';
    return `<div class="fire-row" style="display:grid;grid-template-columns:90px 80px 1fr;gap:10px;padding:8px 10px;border-bottom:1px solid var(--border);font-size:12px;align-items:center;">
      <span style="color:var(--text-dim);font-variant-numeric:tabular-nums;">${escapeHtml(f.firedAt)}</span>
      <span>${outcomeLabel}</span>
      <span>${tail}</span>
    </div>`;
  }).join('');
  return `<div class="panel-block">
    ${panelHeading(heading, null)}
    <div style="border:1px solid var(--border);border-radius:6px;overflow:hidden;background:white;">
      <div class="fire-row head" style="display:grid;grid-template-columns:90px 80px 1fr;gap:10px;padding:6px 10px;background:var(--bg-alt);font-size:10px;text-transform:uppercase;letter-spacing:0.04em;color:var(--text-dim);font-weight:700;border-bottom:1px solid var(--border);">
        <span>When</span>
        <span>Outcome</span>
        <span>${mode === 'failed' ? 'Reason' : 'Spawned'}</span>
      </div>
      ${rows}
    </div>
  </div>`;
}
```

- [ ] **Step 4: Verify in browser**

Open `runtime-prototype.html`.

- Triggers list → click `cron_retention`: expect 3 tabs (Configuration / Recent fires / Failures), Configuration shows schedule `0 2 * * *`, spawns `retention_v2 def_retention_v2`, audit fields populated. Recent fires shows 2 rows (success at 14:02 + failed at 8h ago). Failures shows the failed one with "Open incident →" linking to `inc_006`.
- Triggers list → click `cron_kpi`: expect 2 tabs only (no Failures). Recent fires shows 1 success row.
- Triggers list → click `webhook_sf_sync`: 3 tabs, status pill reads "erroring" (red), Failures shows the invalid-signature fire linking to `inc_003`.
- Triggers list → click `cron_cleanup`: 2 tabs, Recent fires shows "No fires recorded for this trigger yet." (no entries in `triggerFires` for this id).
- Instance `inst_55a1c0` → Overview → click "Loans API webhook" → opens `webhook_loans_api` panel correctly (no more broken link).
- Incident `inc_003` → source-entity link → opens `webhook_sf_sync` panel.
- Esc closes the panel.

- [ ] **Step 5: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): full Trigger detail panel (Configuration · Recent fires · Failures) — fix-list 6"
```

---

## Task 2: Task detail panel — Overview / Form / History + Complete flow

**Files:**
- Modify: `runtime-prototype.html` (MOCK_DATA.tasks entries; new `completeTask` helper; replace `renderTaskDetail` stub; tweak `renderTasks` to re-render the panel when claim/complete happens)

**Closes:** Other half of fix-list Tier 2 #6 (Task detail).

- [ ] **Step 1: Add `createdAt` to each task**

Find the `MOCK_DATA.tasks` block (around line 2076). For each task, append a `createdAt` field. Replace the whole `tasks: [...]` block. Target:

```
  tasks: [
    { id: 'task_1', title: 'Approve loan application — $42,000', source: { instanceId: 'inst_55a1c0', definitionName: 'Loan application', atStep: 'waiting at Manager approval' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'in 4h', isOverdue: false, priority: 'high' },
    { id: 'task_2', title: 'Verify identity documents — passport + utility bill', source: { instanceId: 'inst_19f3d4', definitionName: 'KYC review', atStep: 'awaiting agent' }, assignee: null, dueAt: 'overdue 2h', isOverdue: true, priority: 'high' },
    { id: 'task_3', title: 'Confirm escalation — high-value support case', source: { instanceId: 'sess_b34f0e', definitionName: 'Support chat session', atStep: 'awaiting confirmation' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'due tomorrow', isOverdue: false },
    { id: 'task_4', title: 'Review payment refund — $199.99', source: { instanceId: 'inst_b73e1c', definitionName: 'Payment retry', atStep: 'awaiting review' }, assignee: null, dueAt: 'in 12h', isOverdue: false },
    { id: 'task_5', title: 'Sign agreement', source: { instanceId: 'inst_e1a234', definitionName: 'Document signature', atStep: 'awaiting customer signature' }, assignee: { userId: 'other', displayName: 'CD' }, dueAt: 'in 3 days', isOverdue: false },
    { id: 'task_6', title: 'Risk decision — case r_3344', source: { instanceId: 'inst_a5b6c7', definitionName: 'Risk assessment', atStep: 'awaiting decision' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'in 22h', isOverdue: false },
    { id: 'task_7', title: 'Manual override review', source: { instanceId: 'inst_91ab23', definitionName: 'Email verification', atStep: 'completed' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'completed', isOverdue: false, completed: true },
    { id: 'task_8', title: 'Customer escalation review', source: { instanceId: 'sess_c12d40', definitionName: 'Application form', atStep: 'awaiting review' }, assignee: null, dueAt: 'in 8h', isOverdue: false },
  ],
```

Replace with (adds `createdAt` on each row; nothing else changes):

```
  tasks: [
    { id: 'task_1', title: 'Approve loan application — $42,000', source: { instanceId: 'inst_55a1c0', definitionName: 'Loan application', atStep: 'waiting at Manager approval' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'in 4h', isOverdue: false, priority: 'high', createdAt: '12m ago' },
    { id: 'task_2', title: 'Verify identity documents — passport + utility bill', source: { instanceId: 'inst_19f3d4', definitionName: 'KYC review', atStep: 'awaiting agent' }, assignee: null, dueAt: 'overdue 2h', isOverdue: true, priority: 'high', createdAt: '3h ago' },
    { id: 'task_3', title: 'Confirm escalation — high-value support case', source: { instanceId: 'sess_b34f0e', definitionName: 'Support chat session', atStep: 'awaiting confirmation' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'due tomorrow', isOverdue: false, createdAt: '28m ago' },
    { id: 'task_4', title: 'Review payment refund — $199.99', source: { instanceId: 'inst_b73e1c', definitionName: 'Payment retry', atStep: 'awaiting review' }, assignee: null, dueAt: 'in 12h', isOverdue: false, createdAt: '2h ago' },
    { id: 'task_5', title: 'Sign agreement', source: { instanceId: 'inst_e1a234', definitionName: 'Document signature', atStep: 'awaiting customer signature' }, assignee: { userId: 'other', displayName: 'CD' }, dueAt: 'in 3 days', isOverdue: false, createdAt: '4h ago' },
    { id: 'task_6', title: 'Risk decision — case r_3344', source: { instanceId: 'inst_a5b6c7', definitionName: 'Risk assessment', atStep: 'awaiting decision' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'in 22h', isOverdue: false, createdAt: '6h ago' },
    { id: 'task_7', title: 'Manual override review', source: { instanceId: 'inst_91ab23', definitionName: 'Email verification', atStep: 'completed' }, assignee: { userId: 'me', displayName: 'AB' }, dueAt: 'completed', isOverdue: false, completed: true, createdAt: '1d ago' },
    { id: 'task_8', title: 'Customer escalation review', source: { instanceId: 'sess_c12d40', definitionName: 'Application form', atStep: 'awaiting review' }, assignee: null, dueAt: 'in 8h', isOverdue: false, createdAt: '8h ago' },
  ],
```

- [ ] **Step 2: Add `completeTask` global helper and update `claimTask` to also re-render an open detail panel**

Find the existing `claimTask` (line ~4172):

```js
function claimTask(id) {
  const t = MOCK_DATA.tasks.find(x => x.id === id);
  if (!t) return;
  t.assignee = { userId: 'me', displayName: 'AB' };
  renderTasks(document.getElementById('page'));
}
```

Replace with:

```js
function claimTask(id) {
  const t = MOCK_DATA.tasks.find(x => x.id === id);
  if (!t) return;
  t.assignee = { userId: 'me', displayName: 'AB' };
  renderTasks(document.getElementById('page'));
  if (STATE.detail.open && STATE.detail.kind === 'task' && STATE.detail.id === id) renderDetail();
}
function completeTask(id) {
  const t = MOCK_DATA.tasks.find(x => x.id === id);
  if (!t) return;
  t.completed = true;
  if (!t.assignee) t.assignee = { userId: 'me', displayName: 'AB' };
  renderTasks(document.getElementById('page'));
  if (STATE.detail.open && STATE.detail.kind === 'task' && STATE.detail.id === id) {
    STATE.detail.tab = 2;  // History tab
    renderDetail();
  }
}
```

- [ ] **Step 3: Add `taskFormFields(task)` synthesizer**

Add a new helper immediately above `renderTaskDetail` (which lives at line ~5080):

```js
function taskFormFields(task) {
  const title = (task.title || '').toLowerCase();
  if (title.includes('loan')) {
    return [
      { label: 'Decision', name: 'decision', type: 'select', options: ['Approve', 'Reject', 'Send back for review'] },
      { label: 'Amount approved (USD)', name: 'amount', type: 'text', placeholder: '42000' },
      { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Add reasoning visible to the applicant…' },
    ];
  }
  if (title.includes('verify identity') || title.includes('kyc')) {
    return [
      { label: 'Identity verified', name: 'verified', type: 'select', options: ['Yes', 'No', 'Needs more documents'] },
      { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Anything unusual to flag?' },
    ];
  }
  if (title.includes('refund')) {
    return [
      { label: 'Decision', name: 'decision', type: 'select', options: ['Approve refund', 'Deny refund', 'Partial refund'] },
      { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Customer-visible notes…' },
    ];
  }
  if (title.includes('risk')) {
    return [
      { label: 'Risk level', name: 'risk', type: 'select', options: ['Low', 'Medium', 'High', 'Block'] },
      { label: 'Rationale', name: 'notes', type: 'textarea', placeholder: 'Explain the decision…' },
    ];
  }
  if (title.includes('escalation')) {
    return [
      { label: 'Escalate to', name: 'tier', type: 'select', options: ['Tier 2', 'Tier 3 / specialist', 'Manager'] },
      { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Why escalating?' },
    ];
  }
  if (title.includes('sign agreement')) {
    return [
      { label: 'Signed', name: 'signed', type: 'select', options: ['Signed', 'Declined'] },
      { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Optional comments…' },
    ];
  }
  // Generic fallback
  return [
    { label: 'Decision', name: 'decision', type: 'select', options: ['Approve', 'Reject'] },
    { label: 'Notes', name: 'notes', type: 'textarea', placeholder: 'Add notes…' },
  ];
}
```

- [ ] **Step 4: Replace the `renderTaskDetail` stub**

Find:

```js
function renderTaskDetail(it, tab) {
  return [`<div class="topline"><span class="crumb">Task</span><span style="font-family:'JetBrains Mono',monospace">${escapeHtml(it.id)}</span><button class="x" onclick="closeDetail()">×</button></div><h3>${escapeHtml(it.title)}</h3><div class="sub">From ${escapeHtml(it.source.definitionName)} · ${escapeHtml(it.source.atStep)}</div>`, '', '<div class="empty"><p>Detail content coming in Task 8.</p></div>'];
}
```

Replace with:

```js
function renderTaskDetail(it, tabIdx) {
  const tabs = ['Overview', 'Form', 'History'];

  const dueLabel = (() => {
    if (!it.dueAt) return '<span style="color:var(--text-muted)">—</span>';
    if (it.completed) return `<span style="color:var(--text-muted)">${escapeHtml(it.dueAt)}</span>`;
    if (it.isOverdue) return `<span style="color:var(--red-700);font-weight:600;">${escapeHtml(it.dueAt)}</span>`;
    if (it.dueAt.includes('in 4h') || it.dueAt.includes('in 1h') || it.dueAt.includes('in 2h') || it.dueAt.includes('in 3h')) {
      return `<span style="color:var(--yellow-700);font-weight:600;">${escapeHtml(it.dueAt)}</span>`;
    }
    return escapeHtml(it.dueAt);
  })();
  const priorityPill = it.priority
    ? `<span style="display:inline-block;padding:1px 8px;border-radius:9999px;background:${it.priority === 'high' ? 'var(--red-50)' : 'var(--bg-alt)'};color:${it.priority === 'high' ? 'var(--red-700)' : 'var(--text-muted)'};font-size:11px;font-weight:600;text-transform:capitalize;">${escapeHtml(it.priority)}</span>`
    : '';
  const completedPill = it.completed
    ? `<span style="display:inline-block;padding:1px 8px;border-radius:9999px;background:var(--green-50);color:var(--green-700);font-size:11px;font-weight:600;">Completed</span>`
    : '';

  const head = `
    <div class="topline">
      <span class="crumb">Task</span>
      <span style="color:var(--border-strong)">/</span>
      <span style="font-family:'JetBrains Mono',monospace">${escapeHtml(it.id)}</span>
      <button class="x" onclick="closeDetail()">×</button>
    </div>
    <h3>${escapeHtml(it.title)}</h3>
    <div class="sub">from <a style="color:var(--blue-500);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11px;" onclick="openDetail('instance','${escapeHtml(it.source.instanceId)}')">${escapeHtml(it.source.instanceId)}</a> · ${escapeHtml(it.source.definitionName)} · ${escapeHtml(it.source.atStep)}</div>
    <div class="meta-row" style="display:flex;gap:8px;align-items:center;margin-top:8px;flex-wrap:wrap;">
      ${priorityPill}
      ${dueLabel}
      ${completedPill}
    </div>`;

  const tabsHtml = tabs.map((t, i) => `<button class="tab ${i === tabIdx ? 'active' : ''}" onclick="STATE.detail.tab = ${i}; renderDetail();">${t}</button>`).join('');

  const active = tabs[tabIdx];
  let body = '';
  if (active === 'Overview') {
    const actionBar = (() => {
      if (it.completed) return '';
      if (!it.assignee) {
        return `<div style="display:flex;gap:8px;margin-bottom:14px;"><button class="btn primary" onclick="claimTask('${escapeHtml(it.id)}')">Claim</button></div>`;
      }
      if (it.assignee.userId === 'me') {
        return `<div style="display:flex;gap:8px;margin-bottom:14px;">
          <button class="btn primary" onclick="STATE.detail.tab = 1; renderDetail();">Open form →</button>
          <button class="btn ghost" disabled title="Reassign — not wired in this prototype">Reassign</button>
        </div>`;
      }
      return `<div style="font-size:12px;color:var(--text-muted);margin-bottom:14px;padding:8px 10px;background:var(--bg-alt);border-radius:6px;">Assigned to ${escapeHtml(it.assignee.displayName)}. Only the assignee can complete this task.</div>`;
    })();
    const assigneeCell = it.assignee
      ? `<span style="display:inline-flex;align-items:center;gap:6px;"><span style="display:inline-flex;width:18px;height:18px;border-radius:50%;background:${it.assignee.userId === 'me' ? 'var(--blue-500)' : 'var(--purple-500)'};color:white;font-size:9px;font-weight:700;align-items:center;justify-content:center;">${escapeHtml(it.assignee.displayName)}</span>${escapeHtml(it.assignee.displayName)}${it.assignee.userId === 'me' ? ' (you)' : ''}</span>`
      : '<span style="color:var(--text-muted)">Unassigned</span>';
    body = `${docBanner('Tasks come from processes.', 'When a Process Instance reaches a human-task node it creates a task here and waits for someone to complete it. Opening the originating instance shows where in the flow this is happening.', 'task-overview')}
      ${actionBar}
      <div class="panel-block">
        ${panelHeading('Details', 'Task')}
        <div class="kv">
          <span>Source</span><span><a style="color:var(--blue-500);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11px;" onclick="openDetail('instance','${escapeHtml(it.source.instanceId)}')">${escapeHtml(it.source.instanceId)}</a> · ${escapeHtml(it.source.definitionName)}</span>
          <span>Step</span><span>${escapeHtml(it.source.atStep)}</span>
          <span>Assignee</span><span>${assigneeCell}</span>
          <span>Due</span><span>${dueLabel}</span>
          <span>Priority</span><span>${priorityPill || '<span style="color:var(--text-muted)">—</span>'}</span>
          <span>Created</span><span>${escapeHtml(it.createdAt || '—')}</span>
        </div>
      </div>`;
  } else if (active === 'Form') {
    const fields = taskFormFields(it);
    const disabled = it.completed || !it.assignee || it.assignee.userId !== 'me';
    const fieldsHtml = fields.map(f => {
      if (f.type === 'select') {
        return `<div style="margin-bottom:12px;">
          <div style="font-size:11px;text-transform:uppercase;letter-spacing:0.04em;color:var(--text-dim);font-weight:700;margin-bottom:4px;">${escapeHtml(f.label)}</div>
          <select ${disabled ? 'disabled' : ''} style="width:100%;font-size:12px;padding:6px 10px;border:1px solid var(--border-strong);border-radius:6px;background:white;">
            <option value="">— Select —</option>
            ${f.options.map(o => `<option value="${escapeHtml(o)}">${escapeHtml(o)}</option>`).join('')}
          </select>
        </div>`;
      }
      if (f.type === 'textarea') {
        return `<div style="margin-bottom:12px;">
          <div style="font-size:11px;text-transform:uppercase;letter-spacing:0.04em;color:var(--text-dim);font-weight:700;margin-bottom:4px;">${escapeHtml(f.label)}</div>
          <textarea ${disabled ? 'disabled' : ''} placeholder="${escapeHtml(f.placeholder || '')}" style="width:100%;min-height:60px;font-family:'Open Sans',sans-serif;font-size:12px;padding:8px 10px;border:1px solid var(--border-strong);border-radius:6px;background:white;resize:vertical;box-sizing:border-box;"></textarea>
        </div>`;
      }
      // text
      return `<div style="margin-bottom:12px;">
        <div style="font-size:11px;text-transform:uppercase;letter-spacing:0.04em;color:var(--text-dim);font-weight:700;margin-bottom:4px;">${escapeHtml(f.label)}</div>
        <input type="text" ${disabled ? 'disabled' : ''} placeholder="${escapeHtml(f.placeholder || '')}" style="width:100%;font-size:12px;padding:6px 10px;border:1px solid var(--border-strong);border-radius:6px;background:white;font-family:'JetBrains Mono',monospace;box-sizing:border-box;">
      </div>`;
    }).join('');
    const footer = disabled
      ? `<div style="font-size:11px;color:var(--text-dim);margin-top:4px;">${it.completed ? 'This task is completed — form is read-only.' : !it.assignee ? 'Claim this task first to fill in the form.' : 'Only the assignee can complete this task.'}</div>`
      : `<div style="display:flex;gap:8px;margin-top:12px;">
          <button class="btn primary" onclick="completeTask('${escapeHtml(it.id)}')">Complete task</button>
          <button class="btn ghost" onclick="closeDetail()">Cancel</button>
        </div>`;
    body = `${docBanner('Forms are mocked.', "Fields shown here are illustrative and don't validate or persist — they demonstrate the surface, not the runtime.", 'task-form')}
      <div class="panel-block">
        ${panelHeading('Form', null)}
        ${fieldsHtml}
        ${footer}
      </div>`;
  } else if (active === 'History') {
    const events = [];
    events.push({ at: it.createdAt || 'earlier', kind: 'created', label: `Task <strong>created</strong> from <a style="color:var(--blue-500);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11px;" onclick="openDetail('instance','${escapeHtml(it.source.instanceId)}')">${escapeHtml(it.source.instanceId)}</a> · ${escapeHtml(it.source.definitionName)}` });
    if (it.assignee) {
      events.push({ at: 'after creation', kind: 'claimed', label: `Claimed by <strong>${escapeHtml(it.assignee.displayName)}</strong>${it.assignee.userId === 'me' ? ' (you)' : ''}` });
    }
    if (it.completed) {
      events.push({ at: 'just now', kind: 'completed', label: 'Task <strong>completed</strong>', bad: false });
    }
    const rows = events.map(e => `<div class="tl-row">
      <div class="when">${escapeHtml(e.at)}</div>
      <div class="dot" aria-hidden="true"></div>
      <div class="lbl">${e.label}</div>
    </div>`).join('');
    body = `<div class="panel-block">
      ${panelHeading('History', null)}
      <div class="timeline">${rows}</div>
    </div>`;
  }

  return [head, tabsHtml, body];
}
```

- [ ] **Step 5: Verify in browser**

- Tasks page → click `task_2` Verify identity (unassigned, overdue). Expect 3 tabs; Overview shows red "overdue 2h" chip, source link `inst_19f3d4`, Claim button.
- Click "Claim". Assignee becomes "AB"; Open form button replaces Claim.
- Click "Open form →". Form tab opens; expect `Identity verified` select + `Notes` textarea, both enabled.
- Click "Complete task". Tab switches to History showing 3 rows (created · claimed · completed). Tasks list (under Pending filter) no longer shows `task_2`. Switch to Completed filter → `task_2` appears.
- Open `task_1` Approve loan (assigned to me). Form has loan-specific fields (Decision · Amount approved · Notes).
- Open `task_5` Sign agreement (assigned to "CD"). Overview shows "Assigned to CD. Only the assignee can complete this task." Form fields are disabled.
- Open `task_7` Manual override review (already completed). Header shows "Completed" pill. Action bar empty. Form fields disabled.
- Click any task → source instance chip → opens the originating Instance panel.
- Doc banners dismiss + stay dismissed within the session.

- [ ] **Step 6: Commit**

```bash
git add runtime-prototype.html
git commit -m "feat(runtime): full Task detail panel (Overview · Form · History) + completeTask — fix-list 6"
```

---

## Task 3: Docs — runtime_before_after + fix-list close

**Files:**
- Modify: `runtime_before_after.html` (one new stat + expand a row)
- Modify: `fix-list.md` (mark Tier 2 #6 DONE)

- [ ] **Step 1: Add a new stat to §1 of `runtime_before_after.html`**

Find the existing last stat (after Task 1 of the earlier work it was "Instance detail tabs 8 → 6"):

```
      <div class="stat">
        <div class="k">Instance detail tabs</div>
        <div class="row">
          <span class="before-v">8</span>
          <span class="arrow">→</span>
          <span class="after-v">6</span>
        </div>
        <div class="delta">Tokens · Subprocesses · Trigger · Notifications dropped as standalone tabs — content folded into Activity tree (subprocess tree + token expand), Overview (initiator KV), and Audit (notification timeline). Variables tab gains an edit affordance + modal scaffold.</div>
      </div>
    </div>
  </section>
```

Replace with:

```
      <div class="stat">
        <div class="k">Instance detail tabs</div>
        <div class="row">
          <span class="before-v">8</span>
          <span class="arrow">→</span>
          <span class="after-v">6</span>
        </div>
        <div class="delta">Tokens · Subprocesses · Trigger · Notifications dropped as standalone tabs — content folded into Activity tree (subprocess tree + token expand), Overview (initiator KV), and Audit (notification timeline). Variables tab gains an edit affordance + modal scaffold.</div>
      </div>
      <div class="stat">
        <div class="k">Detail-panel stubs</div>
        <div class="row">
          <span class="before-v">2</span>
          <span class="arrow">→</span>
          <span class="after-v">0</span>
        </div>
        <div class="delta">Trigger and Task detail panels were literal "coming soon" placeholders. Both now ship as full right-slide panels with tabs — Trigger: Configuration · Recent fires · Failures (conditional); Task: Overview · Form · History with working Claim / Complete flow.</div>
      </div>
    </div>
  </section>
```

- [ ] **Step 2: Expand the Task Manager row in §3**

Find the existing row (currently the last row in the page-by-page mapping table):

```
        <tr>
          <td class="before-cell">
            <div class="name">Task Manager</div>
            <div class="desc">User-task inbox. Tasks list their originating process implicitly. No reverse linkage.</div>
          </td>
          <td class="arrow-cell">→</td>
          <td class="after-cell">
            <span class="change-pill keep">Preserved + linked</span>
            <div class="name">Tasks</div>
            <div class="desc">Same list. Each row carries originating Instance ID with click-through; same task surfaces in the Instance's Activity tree tab. Fully cross-linked.</div>
          </td>
        </tr>
```

Replace with:

```
        <tr>
          <td class="before-cell">
            <div class="name">Task Manager</div>
            <div class="desc">User-task inbox. Tasks list their originating process implicitly. No reverse linkage. No detail panel — clicking a task did nothing useful.</div>
          </td>
          <td class="arrow-cell">→</td>
          <td class="after-cell">
            <span class="change-pill enhance">Enhanced</span>
            <div class="name">Tasks</div>
            <div class="desc">
              Same list. Each row carries originating Instance ID with click-through; same task surfaces in the Instance's Activity tree tab. Fully cross-linked.
              <br><br>
              <strong>Detail panel.</strong> Three tabs — <strong>Overview</strong> (source link + assignee + due + priority + Claim / Open form / Reassign actions), <strong>Form</strong> (synthesized fields per task type — loan tasks get Decision + Amount approved + Notes; KYC tasks get Identity verified + Notes; etc.; disabled when completed or not assigned to you), <strong>History</strong> (created / claimed / completed timeline). Working in-session Complete that moves the row out of Pending into Completed.
            </div>
          </td>
        </tr>
```

- [ ] **Step 3: Add a Trigger panel row to §3**

Inside the same `<tbody>` of the mapping table, immediately AFTER the existing "Scheduled Processes / Manage Triggers → Schedules & Triggers" row, add a new row covering the Trigger detail panel. Target the closing of that row:

```
        <tr>
          <td class="before-cell">
            <div class="name">Scheduled Processes<br>Manage Triggers</div>
            <div class="desc">Mutually linked configuration surfaces. Sibling failure pages (Failed Triggers / Failed Process Start) live elsewhere in the nav.</div>
          </td>
          <td class="arrow-cell">→</td>
          <td class="after-cell">
            <span class="change-pill merge">Merged</span>
            <div class="name">Schedules &amp; Triggers</div>
            <div class="desc">One page, four tabs (Scheduled · External · Recent fires · Failed fires). Trigger-stage failures double-surface in Incidents. Two-word label preserves both mental models — users hunting for "the nightly batch job" find it under <em>Schedules</em>, users wiring a webhook find it under <em>Triggers</em>.</div>
          </td>
        </tr>
```

Replace with the same row PLUS a new follow-up row immediately after:

```
        <tr>
          <td class="before-cell">
            <div class="name">Scheduled Processes<br>Manage Triggers</div>
            <div class="desc">Mutually linked configuration surfaces. Sibling failure pages (Failed Triggers / Failed Process Start) live elsewhere in the nav.</div>
          </td>
          <td class="arrow-cell">→</td>
          <td class="after-cell">
            <span class="change-pill merge">Merged</span>
            <div class="name">Schedules &amp; Triggers</div>
            <div class="desc">One page, four tabs (Scheduled · External · Recent fires · Failed fires). Trigger-stage failures double-surface in Incidents. Two-word label preserves both mental models — users hunting for "the nightly batch job" find it under <em>Schedules</em>, users wiring a webhook find it under <em>Triggers</em>.</div>
          </td>
        </tr>

        <tr>
          <td class="before-cell">
            <div class="name">Trigger detail (none)</div>
            <div class="desc">Clicking a trigger row in the old Manage Triggers / Scheduled Processes pages either did nothing or revealed thin config-only modals. No fire history alongside config. No path to incidents.</div>
          </td>
          <td class="arrow-cell">→</td>
          <td class="after-cell">
            <span class="change-pill new">New</span>
            <div class="name">Trigger detail panel</div>
            <div class="desc">
              Right-slide panel with three tabs — <strong>Configuration</strong> (type · schedule · spawns · status · 24h error count · audit fields), <strong>Recent fires</strong> (chronological history per trigger with spawned-instance links), and <strong>Failures</strong> (conditional; only when the trigger has any failed fires; rows deep-link to the corresponding Incident). Reachable from the Triggers list, from Incident detail's source-entity link when <code>sourceEntity.kind === 'trigger'</code>, and from Instance Overview's Trigger initiator row.
            </div>
          </td>
        </tr>
```

- [ ] **Step 4: Update `fix-list.md` to mark Tier 2 #6 DONE**

Find the existing block:

```
### 6. Trigger and Task detail panels are stubs
**Spec:** §8 — Triggers and Task Manager are first-class surroundings; tasks come from Process Instances reaching human-task nodes.
**Prototype state:** `renderTriggerDetail` (line 3396) and `renderTaskDetail` (line 3399) return literal "Detail content coming in Task 8" placeholders.
**What "fixed" means:** flesh out both detail panels — Trigger needs schedule/event config, recent fires, spawned instances; Task needs source instance link, claim/complete actions, variables snapshot.
```

Replace with:

```
### 6. Trigger and Task detail panels are stubs  ✅ **DONE**
**Spec:** §8 — Triggers and Task Manager are first-class surroundings; tasks come from Process Instances reaching human-task nodes.
**Resolution:** both stubs replaced with full right-slide panels following the same `[head, tabs, body]` triple convention used by `renderInstanceDetail` / `renderIncidentDetail`.
- **Trigger panel** — three tabs (Configuration · Recent fires · Failures). Failures is conditional (rendered only when the trigger has ≥1 failed fire). Configuration shows schedule (mono when cron), spawned definition, status pill, 24h error count, and audit fields. Recent fires lists chronological fires from `MOCK_DATA.triggerFires` filtered by trigger id with click-through to spawned instances on success. Failures rows deep-link to the corresponding Incident via `sourceEntity.kind === 'trigger'`. Adds the missing `webhook_loans_api` trigger so the Instance Overview's cross-link from Tier 2 #5 resolves cleanly.
- **Task panel** — three tabs (Overview · Form · History). Overview surfaces source instance link, assignee, due chip, priority pill, and an action row that adapts: Claim (unassigned) / Open form → (assigned to me) / read-only "Assigned to X" (assigned to another) / completed pill. Form synthesizes 2–3 fields tailored to the task title (loan tasks get Decision + Amount approved + Notes; KYC gets Identity verified + Notes; etc.); disabled when completed or not assigned to you; submit calls `completeTask(id)`. History derives created / claimed / completed events from current state. New `completeTask(id)` matches the existing `claimTask(id)` pattern (direct `MOCK_DATA.tasks` mutation), so the Tasks list auto-correctly moves completed tasks out of the Pending filter.
```

- [ ] **Step 5: Commit**

```bash
git add runtime_before_after.html fix-list.md
git commit -m "docs(runtime): close fix-list 6 — update before/after + DONE markers"
```

---

## Self-review checklist (executed by plan author)

- **Spec coverage:**
  - Trigger 3 tabs (Configuration / Recent fires / Failures conditional) — Task 1 Step 3
  - Trigger header (crumb / id / name / kind icon / status pill / last fire) — Task 1 Step 3
  - Trigger Configuration KV (Type / Schedule / Spawns / Last fire / Status / Errors 24h / Audit ×2) — Task 1 Step 3
  - Trigger Recent fires + Failures tables with incident deep-link — Task 1 Step 3
  - Task 3 tabs (Overview / Form / History) — Task 2 Step 4
  - Task action bar (Claim / Open form → / read-only / completed) — Task 2 Step 4
  - Task Form synthesized fields per title — Task 2 Step 3 + 4
  - Task Complete → mutates MOCK_DATA, re-renders list + switches to History — Task 2 Steps 2 + 4
  - In-session contract via direct MOCK_DATA mutation (noted spec deviation) — plan intro
  - Mock-data additions for triggers (`spawnsDefinitionId` + audit) — Task 1 Step 1
  - New `webhook_loans_api` trigger so cross-link resolves — Task 1 Step 2
  - `createdAt` on tasks — Task 2 Step 1
  - Doc banners (`trigger-config`, `task-overview`, `task-form`) — Task 1 Step 3, Task 2 Step 4
  - Acceptance walkthrough — Tasks 1 & 2 verification steps
  - runtime_before_after.html update — Task 3

- **Placeholder scan:** None. Every step has the full code.

- **Type consistency:** `taskFormFields`, `completeTask`, `claimTask`, `renderTriggerDetail`, `renderTaskDetail`, `renderTriggerFiresList` names consistent across all tasks. `STATE.detail.tab = N` assignments use the right indices per panel (Trigger: 0=Config 1=Recent 2=Failures; Task: 0=Overview 1=Form 2=History) — `completeTask` switches to tab 2 which is History.

- **Scope:** 2 prototype changes + 2 doc changes = 3 commits. Each ends in a working state.
