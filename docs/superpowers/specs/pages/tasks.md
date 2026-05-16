# Tasks — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

The Tasks page is the user-task inbox for the Runtime workspace. Its existing inbox-style behavior is largely preserved: a single list of rows that operators filter by assignee, status, and due window, with a right-slide detail panel for opening, claiming, and completing individual tasks.

What changes in this redesign is the explicit cross-linking with Process Instances. Every row now surfaces its originating Instance ID, the definition name, and the step that produced the task, with click-through into the Instance. The reverse path also holds: the same task appears as a child node of the human-task step inside that Instance's Activity tree (Plan 04), and clicking it there opens the same detail panel used from the inbox.

This bidirectional link is the primary integration point with the rest of Runtime. It lets a business analyst or support user move fluidly between "what work is waiting on a human" and "where in the flow that work is happening", without losing context.

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

## Component API

List props: rows; filters; onRowClick; onClaim; onAssign; onComplete.

Filters: { assignee: 'me'|'unassigned'|'anyone'; status: 'pending'|'completed'; dueWindow: 'today'|'this-week'|'overdue'|'all' }.

URL: `/runtime/tasks?assignee=me&status=pending&due=`. Detail: `…?detail={taskId}` (right-slide) or `/runtime/tasks/{taskId}` (maximized).

Reverse rendering: the Instance Activity tree (Plan 04) renders tasks created by that instance as child nodes of the relevant step. Clicking a task there opens the same detail panel.

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

## Tokens

- Avatar bg (assigned): `color/blue/500`
- Avatar bg (unassigned): `color/border/strong` with `color/text/muted` "?" glyph
- Due overdue label: `color/red/700`, font-weight semibold
- Due-soon label: `color/yellow/700`
- Action button: `color/blue/500` bg, white text
- Source link: monospace ID chip + lighter-weight definition name

## Open questions for engineering

Items: form rendering engine (re-use Designer form runtime?), assignment workflow (manual claim vs auto-assign rules), notifications when a task is assigned, SLA/escalation logic, reverse-render performance in Activity tree.

## Figma + user testing — pending

- [ ] Build hi-fi static mockup: "Tasks / Inbox" frame with 3–5 rows including one overdue (red due label), one unassigned (grey avatar), each showing origin instance ID.
- [ ] Build hi-fi static mockup: "Tasks / Detail (right-slide)" frame showing task form, history, and the originating Instance link prominently in the header meta-row.
- [ ] Add Figma variants for each interaction state in the matrix above.
- [ ] User test script — 5 tasks: (a) find tasks assigned to you and open one, (b) claim an unassigned task, (c) navigate from a task to its originating instance, (d) navigate from an instance's Activity tree to a task it created, (e) filter to overdue tasks.
- [ ] Run user test with 5 participants, emphasizing the business analyst / support persona.
- [ ] Capture findings under a `## User test summary` section and iterate before marking "Ready for engineering".
