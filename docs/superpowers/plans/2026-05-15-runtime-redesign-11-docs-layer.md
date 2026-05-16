# Documentation Layer — Plan 11

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the six documentation affordances that make the Runtime tab self-teaching: inline help icons + glossary side panel, contextual doc banners, empty-state explanations, tour mode, glossary, state pill tooltips.

**Architecture:** Cross-cutting concern that ships in parallel with the page plans. Each affordance is a reusable component or pattern, sourced from a single glossary catalog and applied consistently.

**Deliverables:**
- Create: `docs/superpowers/specs/components/help-popover.md`
- Create: `docs/superpowers/specs/components/doc-banner.md`
- Create: `docs/superpowers/specs/components/empty-state.md`
- Create: `docs/superpowers/specs/components/tour.md`
- Create: `docs/superpowers/specs/components/glossary-panel.md`
- Create: `docs/superpowers/specs/runtime-glossary.md` — the single source-of-truth catalog
- Figma: variants for each affordance

---

### Task 1: Build the glossary catalog

**Files:** create `docs/superpowers/specs/runtime-glossary.md`

- [ ] **Step 1: Write canonical definitions for every concept**

```markdown
## Runtime glossary

### Build
An immutable, published snapshot of an application. Carries all process definitions, UI flows, workflows, AI nodes, and config params for that snapshot. Each Build has a tag (e.g. `v1.4.2`), a branch it was cut from, and a build date.

### Branch
A line of build evolution. Most apps have at least a `main` branch; many have `staging`, `beta`, or feature branches. A policy can track a branch's latest build or pin to a specific build.

### Active policy
The default rule for picking which Build serves a runtime caller. Either `LATEST_ON_BRANCH` (track the tip of a branch) or `FIXED_VERSION` (pin to a specific Build).

### Policy override
A rule that replaces the default policy for a specific user or role. Has a priority; higher priority wins when multiple overrides match. Can be toggled off without deletion.

### Config parameter
A configurable value the running app consumes (e.g. `max_retries`, `llm_temperature`, `region`). Has a default; per-user/role overrides work like policy overrides.

### Process Instance
One execution of a process definition. Has state (running/waiting/completed/failed/terminated), tokens at active nodes, variables, and optionally sub-processes.

### Token
A pointer inside a Process Instance marking a currently-active node. A process can have multiple tokens running in parallel branches.

### UI Flow Session
One user-facing execution of a UI flow — chat, conversational UI, multi-step form. Can spawn Process Instances as sub-work.

### Incident
Something that broke or breached. Four stages: pre-start (process refused to start), trigger (trigger fired but didn't start a process), mid-execution (process threw mid-run), AI quality (an AI scorer crossed its threshold).

### Trigger
A configured way for processes to start. Cron triggers fire on a schedule; webhooks/events fire on external signals.

### Live Evals
The runtime observability surface for AI workflow quality — telemetry (free), workflow scorers (opt-in), and AI node detail.

### Scorer
A configured quality measurement that runs against AI workflow output. Has a threshold; crossing the threshold creates an AI quality incident.

### Task
A unit of human work created when a Process Instance reaches a human-task node. The instance waits until the task is completed.

### Normalized status
The simplified five-state vocabulary the UI uses for all instance types: Running, Waiting, Completed, Failed, Terminated. Hover any status pill to see the underlying engineering state.
```

- [ ] **Step 2: Commit**: `spec: runtime glossary catalog`

---

### Task 2: Define HelpPopover component

**Files:** create `docs/superpowers/specs/components/help-popover.md`

- [ ] **Step 1: Document API and behavior**

```markdown
## HelpPopover

Props: term: keyof Glossary; short?: string; placement?: 'top'|'bottom'|'left'|'right'.

Rendered as a small `?` icon inline. On hover/focus, opens a popover with: the glossary `short` (1-line) by default; a "Read more →" link that opens the GlossaryPanel scrolled to that term.

When clicked (not hovered), the popover pins and the popover can take keyboard focus for screen readers.

Empty popover content fallback: "No definition yet — contribute one in the runtime-glossary.md doc."
```

- [ ] **Step 2: Figma variant**

Frame "HelpPopover / Inline icon + Open popover"

- [ ] **Step 3: Commit**: `spec: help-popover component`

---

### Task 3: Define DocBanner component

**Files:** create `docs/superpowers/specs/components/doc-banner.md`

- [ ] **Step 1: Document API and dismissal logic**

```markdown
## DocBanner

Props: id: string (stable across versions); title: string; body: string; dismissible?: boolean = true; severity?: 'info'|'warn' = 'info'.

Dismissal: when user clicks ×, persist `dismissed:{id}:{userId}:{contentHash}`. Re-show only when contentHash changes (i.e. when the banner copy meaningfully changes).

Placement: above panel content or inside a tab body. The yellow severity is the default; warn variant uses orange.

Accessibility: `role="region" aria-label={title}"`; dismiss button has `aria-label="Dismiss this help"`.
```

- [ ] **Step 2: Figma variant**

Frame "DocBanner / Default · Dismissed"

- [ ] **Step 3: Commit**: `spec: doc-banner component`

---

### Task 4: Define EmptyState component

**Files:** create `docs/superpowers/specs/components/empty-state.md`

- [ ] **Step 1: Document API and copy patterns**

```markdown
## EmptyState

Props: title: string; body: string; cta?: { label: string; onClick }; tone?: 'neutral'|'positive' = 'neutral'.

Tone:
- 'neutral' — when an absence is expected ("No active instances yet…")
- 'positive' — when an absence is good ("All clear — no open incidents.")

Pattern enforced across all pages: empty states must explain (a) what would be here, and (b) how it gets here.

Examples (canonical copy per page):
- Instances (no filter): "No active instances yet — when a UI flow runs or a process starts, it'll appear here."
- Incidents (all clear): "All clear — no open incidents."
- Tasks (none for me): "No tasks for you right now. New tasks from running processes will appear here."
- Triggers (none scheduled): "No scheduled processes yet — schedule a process to run on a cadence."
- Configuration (no overrides): "No overrides yet — all callers get the default. Add an override to differentiate a user or role."
```

- [ ] **Step 2: Figma variants**

Frames for neutral and positive tones.

- [ ] **Step 3: Commit**: `spec: empty-state component`

---

### Task 5: Define Tour mode

**Files:** create `docs/superpowers/specs/components/tour.md`

- [ ] **Step 1: Document tour controller and per-page tour spec**

```markdown
## Tour

Controller props: pageId: string; steps: TourStep[]; autoRun: 'first-visit'|'never'|'always-prompt'.

interface TourStep { selector: string; title: string; body: string; placement?: 'top'|'bottom'|'left'|'right' }.

Behavior:
- Footer "Tour & help" link in sidebar always launches the current page's tour on demand.
- Overview tour: autoRun 'first-visit' for new users (stored as `tour:overview:done:{userId}`).
- Every other page: autoRun 'never'; available on demand.
- Each step is 1-2 sentences max, dismiss anytime, "Next" / "Skip tour" controls.

Initial tours (sequence of steps for each page) — copy lives in the page's own spec doc. This plan owns the controller; per-page tour content is owned by the page plan's copy task.
```

- [ ] **Step 2: Figma variant**

Frame "Tour / Overview / Step 1 of 4" with overlay + highlighted target.

- [ ] **Step 3: Commit**: `spec: tour controller`

---

### Task 6: Define GlossaryPanel component

**Files:** create `docs/superpowers/specs/components/glossary-panel.md`

- [ ] **Step 1: Document panel behavior**

```markdown
## GlossaryPanel

Right-slide panel (same chrome as Plan 01 detail-panel for visual consistency) listing every term in `runtime-glossary.md`. Reachable from:
- Any HelpPopover's "Read more →" link (scrolls to specific term)
- A "?" button in the sidebar footer (opens panel at top)
- Cmd+K (or `?` keyboard shortcut) global hotkey

Props: open: boolean; scrollToTerm?: string; onClose.

Implementation: panel reads the same glossary file used by HelpPopover — single source of truth.

Search box at the top filters terms in real-time.
```

- [ ] **Step 2: Figma variant**

Frame "GlossaryPanel / Open" showing term list with one term highlighted as the scroll target.

- [ ] **Step 3: Commit**: `spec: glossary-panel component`

---

### Task 7: Define state pill tooltip pattern

**Files:** modify `docs/superpowers/specs/components/empty-state.md` ← actually create `docs/superpowers/specs/components/state-pill.md`

- [ ] **Step 1: Document the pill component**

```markdown
## StatePill

Props: state: NormalizedStatus; rawState: string; rawDescription?: string.

Renders a colored pill with the normalized label (Running / Waiting / Completed / Failed / Terminated). On hover (and on keyboard focus for accessibility), shows tooltip:

"{rawState} · {rawDescription}"

Examples:
- state: 'failed', rawState: 'FINISHED_WITH_ERROR', rawDescription: 'raised exception during execution'
- state: 'waiting', rawState: 'ON_HOLD', rawDescription: 'paused awaiting input'

Mapping table (see Plan 04 information model) is the source of truth for which rawStates map to which normalized states and which descriptions accompany them.
```

- [ ] **Step 2: Figma variant**

Frame "StatePill / Default + Hover tooltip" for all 5 states.

- [ ] **Step 3: Commit**: `spec: state-pill component`

---

### Task 8: User test the documentation layer end-to-end

- [ ] **Step 1: Script**

7 tasks across pages: (a) find the definition of "Build" without leaving the current page, (b) dismiss a doc banner and verify it stays dismissed, (c) launch the Overview tour and complete it, (d) open the glossary, search for "scorer", read it, (e) hover a "Failed" status pill and explain what the raw state means, (f) interpret an empty state on the Tasks page and explain how a task gets there, (g) navigate from a doc banner's "Read more" link to the glossary.

- [ ] **Step 2: Run with 6-8 participants spanning all three personas, including at least one user new to FlowX**

- [ ] **Step 3: Capture findings**

Goal: every concept should be reachable + understandable within 30 seconds of seeing it for the first time.

- [ ] **Step 4: Write `## User test summary`** across each affordance's spec doc

- [ ] **Step 5: Commit**: `spec(docs-layer): cross-cutting user-test summary`

---

### Task 9: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: Add `## Open questions for engineering`** to each component spec

Common items: i18n/localization strategy for glossary terms, dismissal persistence storage (localStorage vs server), tour library choice, popover library, keyboard nav details, RTL support, screen-reader announcements.

- [ ] **Step 3: Mark each component "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(docs-layer): handoff`
