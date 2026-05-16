# Triggers Page — Plan 09

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the combined Triggers page — Scheduled Processes + Manage Triggers + their failure history under one nav entry.

**Architecture:** Four tabs: Scheduled · External · Recent fires · Failed fires. Trigger-stage failures double-surface in Incidents; this page provides the trigger-centric view.

**Deliverables:**
- Create: `docs/superpowers/specs/pages/triggers.md`
- Figma: "Triggers / Scheduled", "… / External", "… / Recent fires", "… / Failed fires", "… / Detail (any tab)"

**Depends on:** Plan 01, Plan 02.

---

### Task 1: Define information model

- [ ] **Step 1: Document data**

```markdown
## Information model

interface Trigger {
  id: string;
  name: string;
  type: 'cron' | 'webhook' | 'event';
  schedule?: string;             // cron expression or webhook description
  spawnsDefinitionName: string;
  spawnsDefinitionId: string;
  lastFireAt?: string;
  status: 'active' | 'paused' | 'erroring';
  errorCount24h: number;
  audit: AuditFields;
}

interface TriggerFire {
  id: string;
  triggerId: string;
  triggerName: string;
  triggerType: Trigger['type'];
  firedAt: string;
  outcome: 'success' | 'failed';
  spawnedInstanceId?: string;     // only when outcome === 'success'
  failureReason?: string;         // only when outcome === 'failed'
}

interface TriggerDetail extends Trigger {
  recentFires: TriggerFire[];
  failuresLast7d: TriggerFire[];
}
```

- [ ] **Step 2: Commit**: `spec: triggers information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document props and URL contract**

```markdown
## Component API

Page props: tabs ('scheduled'|'external'|'recent-fires'|'failed-fires'), rows, filters, onRowClick, onPauseToggle, onTabChange.

URL: `/runtime/triggers?tab=scheduled&q=&status=`. Detail: `…?detail={triggerId}` (right-slide panel) or `/runtime/triggers/{triggerId}` (maximized).

Tab content:
- Scheduled tab: Trigger rows where type === 'cron', columns: Name · Type · Schedule · Spawns · Last fire · Status (active/paused with toggle)
- External tab: Trigger rows where type !== 'cron', columns same but Schedule → "on event"/"webhook"
- Recent fires tab: TriggerFire rows across all triggers, columns: Trigger name · Type · Fired at · Outcome · Spawned instance link
- Failed fires tab: TriggerFire rows where outcome === 'failed', columns same as Recent + Failure reason

A failed fire in this list double-surfaces in Incidents page (stage: trigger). Clicking a failed-fire here can deep-link to the Incidents detail for the same record if it has been escalated.
```

- [ ] **Step 2: Commit**: `spec: triggers component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: Scheduled tab**

Figma "Triggers / Scheduled" with 3-4 cron rows including one paused and one with active failures (red-bg row).

- [ ] **Step 2: External tab**

Frame: webhooks and events, including the Salesforce sync row with active failures.

- [ ] **Step 3: Recent fires + Failed fires tabs**

Frames showing time-ordered fire history with outcome columns.

- [ ] **Step 4: Commit**: `spec(triggers): tab mockups`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Active row | status === 'active' | Green pill "Active" |
| Paused row | status === 'paused' | Grey pill "Paused" |
| Erroring row | errorCount24h > 0 | Row bg `color/red/50`, red pill with count |
| Pause toggle | Click switch | Optimistic update, error toast on failure |
| Tab switch | Click tab | URL updates, content rerenders |
| Empty Scheduled | No cron triggers | "No scheduled processes yet — schedule a process to run on a cadence." |
| Empty External | No external triggers | "No external triggers configured — set up a webhook or event subscription." |
| Empty Failed fires | No failures in window | "No trigger failures in {window} — triggers are healthy." (positive empty) |
```

- [ ] **Step 2: Add Figma variants**

- [ ] **Step 3: Commit**: `spec(triggers): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document copy**

```markdown
## Copy

- Page title: "Triggers"
- Help icon: "Scheduled processes + external triggers + their failure history, one page."
- Tab labels: "Scheduled", "External", "Recent fires", "Failed fires"
- Column headers: "Name", "Type", "Schedule", "Spawns", "Last fire", "Status"
- Type labels: "Cron", "Webhook", "Event"
- Status pills: "Active", "Paused", "{n} failure(s) (active)"
- Outcome labels: "Success", "Failed"
- Doc banner (cross-page note): "What's combined here. Old Scheduled Processes + old Manage Triggers + the trigger-failure subset of Failed Triggers. Mid-execution / Pre-start failures live in Incidents. The 'Failed fires' tab here is just for fires that didn't reach a process start."
- Empty Scheduled: "No scheduled processes yet — schedule a process to run on a cadence."
- Empty External: "No external triggers configured — set up a webhook or event subscription."
- Empty Failed fires: "No trigger failures in {window} — triggers are healthy."
- Pause confirmation: "Pause this trigger? It won't fire until you resume."
- Detail tabs: "Configuration", "Recent fires", "Failures"
```

- [ ] **Step 2: Commit**: `spec(triggers): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: Token list**

```markdown
## Tokens

- Active status pill: `color/green/50` / `color/green/700`
- Paused status pill: `color/surface/alt` / `color/text/muted`
- Erroring row bg: `color/red/50`
- Erroring status pill: `color/red/50` / `color/red/700`
- Cron schedule cell: monospace (`font/code/xs`)
- Outcome success: `color/green/700`
- Outcome failed: `color/red/700`
```

- [ ] **Step 2: Commit**: `spec(triggers): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Script**

5 tasks: (a) find a scheduled process and pause it, (b) tell me why a trigger is erroring, (c) view recent fires for "Salesforce sync", (d) navigate from a failed fire to its Incident record, (e) tell me which trigger spawned a given instance.

- [ ] **Step 2: Run with 5 participants (emphasis on ops + admin)**

- [ ] **Step 3: `## User test summary`**

- [ ] **Step 4: Commit**: `spec(triggers): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: `## Open questions for engineering`**

Items: pause/resume API contract, webhook signature verification UX, replay-failed-fire affordance (future), how Failed fires here are linked to Incidents records.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(triggers): handoff`
