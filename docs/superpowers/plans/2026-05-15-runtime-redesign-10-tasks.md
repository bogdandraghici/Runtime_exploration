# Tasks Page — Plan 10

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the Tasks page — largely preserved, but with explicit cross-linking to originating Instances and rendering in the Instance's Activity tree tab.

**Architecture:** Single inbox-style list. Each row carries originating Instance ID with click-through. The same task surfaces in the Instance's Activity tree.

**Deliverables:**
- Create: `docs/superpowers/specs/pages/tasks.md`
- Figma: "Tasks / Inbox", "Tasks / Detail (right-slide)"

**Depends on:** Plan 01, Plan 02, Plan 04 (for the reverse-render in Activity tree).

---

### Task 1: Define information model

- [ ] **Step 1: Document data**

```markdown
## Information model

interface TaskRow {
  id: string;
  title: string;
  source: { instanceId: string; instanceType: 'process'|'uiflow'; definitionName: string; atStep: string };
  assignee?: { userId: string; displayName: string; avatarUrl?: string };  // undefined → unassigned
  dueAt?: string;
  isOverdue: boolean;
  priority?: 'low'|'medium'|'high';
  createdAt: string;
}

interface TaskDetail extends TaskRow {
  form: TaskForm;                 // the actual form fields the task is collecting
  history: TaskHistoryEvent[];    // claims, re-assignments, comments
}
```

- [ ] **Step 2: Commit**: `spec: tasks information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document props and URL contract**

```markdown
## Component API

List props: rows; filters; onRowClick; onClaim; onAssign; onComplete.

Filters: { assignee: 'me'|'unassigned'|'anyone'; status: 'pending'|'completed'; dueWindow: 'today'|'this-week'|'overdue'|'all' }.

URL: `/runtime/tasks?assignee=me&status=pending&due=`. Detail: `…?detail={taskId}` (right-slide) or `/runtime/tasks/{taskId}` (maximized).

Reverse rendering: the Instance Activity tree (Plan 04) renders tasks created by that instance as child nodes of the relevant step. Clicking a task there opens the same detail panel.
```

- [ ] **Step 2: Commit**: `spec: tasks component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: List mockup**

Figma "Tasks / Inbox" — 3-5 rows including one overdue (red due label), one unassigned (grey avatar), each with origin instance ID shown.

- [ ] **Step 2: Detail mockup**

Frame showing the task form, history, and the originating Instance link prominently in the header meta-row.

- [ ] **Step 3: Commit**: `spec(tasks): list + detail mockups`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Assigned to me | task.assignee.userId === currentUser.id | Default row |
| Unassigned | task.assignee === undefined | Grey "?" avatar; "Claim" button |
| Overdue | task.isOverdue | Red due label "overdue {duration}" |
| Due soon | dueAt within 4h | Yellow due label |
| Claim action | Click Claim | Optimistic: avatar replaces grey, button changes to "Open" |
| Complete action | Form submit on detail | Row leaves "pending" filter view, instance's Activity tree updates the step |
| Empty pending for me | filtered no results | "No tasks for you right now. New tasks from running processes will appear here." |
```

- [ ] **Step 2: Add Figma variants**

- [ ] **Step 3: Commit**: `spec(tasks): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document copy**

```markdown
## Copy

- Page title: "Tasks"
- Help icon: "User-task inbox. Tasks are created by Process Instances reaching a human-task node."
- Filter labels: "Assignee", "Status", "Due"
- Assignee options: "Me", "Unassigned", "Anyone"
- Status options: "Pending", "Completed"
- Due options: "Today", "This week", "Overdue", "All"
- Action buttons: "Open" (when assigned to current user), "Claim" (when unassigned), "View" (when assigned to others)
- Due labels: "due in {duration}", "due tomorrow", "due {date}", "overdue {duration}"
- Source label: "from {instanceId} · {definitionName} · {status} at {step}"
- Empty (pending for me): "No tasks for you right now. New tasks from running processes will appear here."
- Empty (filtered): "No tasks match these filters."
- Detail crumb: "Task"
- Doc banner: "Tasks come from processes. When a process reaches a human-task node, it creates a task here and waits for someone to complete it. Opening the originating instance shows where in the flow this is happening."
```

- [ ] **Step 2: Commit**: `spec(tasks): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: Token list**

```markdown
## Tokens

- Avatar bg (assigned): `color/blue/500`
- Avatar bg (unassigned): `color/border/strong` with `color/text/muted` "?" glyph
- Due overdue label: `color/red/700`, font-weight semibold
- Due-soon label: `color/yellow/700`
- Action button: `color/blue/500` bg, white text
- Source link: monospace ID chip + lighter-weight definition name
```

- [ ] **Step 2: Commit**: `spec(tasks): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Script**

5 tasks: (a) find tasks assigned to you and open one, (b) claim an unassigned task, (c) navigate from a task to its originating instance, (d) navigate from an instance's Activity tree to a task it created, (e) filter to overdue tasks.

- [ ] **Step 2: Run with 5 participants (emphasize business analyst / support persona)**

- [ ] **Step 3: `## User test summary`**

- [ ] **Step 4: Commit**: `spec(tasks): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: `## Open questions for engineering`**

Items: form rendering engine (re-use Designer form runtime?), assignment workflow (manual claim vs auto-assign rules), notifications when a task is assigned, SLA/escalation logic, reverse-render performance in Activity tree.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(tasks): handoff`
