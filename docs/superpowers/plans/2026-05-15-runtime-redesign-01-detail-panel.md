# Shared Detail-Panel Component — Plan 01

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify a single reusable right-slide detail-panel component that all entity detail views (Instance, Incident, Build, Trigger, Task) consume. This is the highest-leverage foundation piece — resolves friction #8.

**Architecture:** One component with a strict API. Tabbed body, header with crumb/title/sub/meta/actions, optional doc-banner slot, panels (`.panel` blocks) for content. Default width 460–480px right-slide, maximizable to full route, closeable to URL state.

**Tech Stack:** Figma, FlowX DS tokens, Markdown spec doc.

**Deliverables:**
- Create: `docs/superpowers/specs/components/detail-panel.md`
- Create: Figma frame "Runtime / Detail panel" with default + all interaction states

---

### Task 1: Define information model

**Files:** create `docs/superpowers/specs/components/detail-panel.md`

- [ ] **Step 1: Document the data the panel consumes**

```markdown
## Information model

interface DetailPanelData {
  // Header
  crumb: string;             // e.g. "Instance details"
  typedId: string;           // monospace ID e.g. "sess_a82c91"
  title: string;             // e.g. "Onboarding chat"
  subline: string;           // e.g. "UI Flow session · started 1m ago"
  statusPill?: StatusPill;   // normalized state pill
  metaChips: MetaChip[];     // build chip, step indicator, severity, etc.
  primaryActions: Action[];  // up to 3 buttons (Acknowledge, Open, Compare, …)
  // Body
  tabs: Tab[];               // 2–7 tabs, each owns its own content area
  docBanner?: DocBanner;     // dismissable yellow banner at top of body
  // State
  initialTab?: string;       // which tab is open on mount
  maximized?: boolean;       // full-route vs right-slide
}

interface Tab { id: string; label: string; count?: number; countAlert?: boolean; content: ReactNode; }
```

- [ ] **Step 2: Commit**

Commit message: `spec: define detail-panel information model`

---

### Task 2: Define component API

**Files:** modify `docs/superpowers/specs/components/detail-panel.md`

- [ ] **Step 1: Document props, events, and URL contract**

```markdown
## Component API

Props: data: DetailPanelData; open: boolean; onClose: () => void; onMaximize: () => void; onTabChange: (id: string) => void;

URL contract:
- Right-slide mode: parent route owns the URL, panel state mirrors `?detail=<entityId>&tab=<tabId>`
- Maximized mode: dedicated route `<parent>/<entityId>/<tabId>`

Behaviors:
- Esc key → onClose
- Cmd/Ctrl + click on a tab → opens that tab in maximized mode (new context)
- Sticky header (title + tabs) when body scrolls
- Doc banner dismissal persists per-user, per-component-type, per-tab
```

- [ ] **Step 2: Commit**

Commit message: `spec: define detail-panel component API`

---

### Task 3: Build hi-fi static mockup in Figma

- [ ] **Step 1: Create Figma frame**

Frame: "Runtime / Detail panel / Default state".
Content: header with all elements, 4 tabs, doc banner, two example `.panel` blocks (one kv pair list, one custom content slot), action buttons.
Breakpoints: 1440px (right-slide 480px) and 1280px (right-slide 460px). Maximized variant at 1440px (full route).

- [ ] **Step 2: Paste Figma frame URL into spec doc**

Add under `## Mockups` section in `detail-panel.md`.

- [ ] **Step 3: Commit**

Commit message: `spec(detail-panel): default-state mockup`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix in spec**

```markdown
## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Default | Panel open with data | As designed |
| Loading | Data fetch in progress | Header skeleton, tabs visible, body shimmer |
| Error | Data fetch failed | Body shows error card with retry button |
| Empty tab | Tab has no content yet | Empty-state explanation (see Plan 11) |
| Closed | onClose triggered | Slide-out animation 200ms ease-out |
| Maximized | onMaximize | Animate to full route over 300ms |
| Tab switch | onTabChange | Crossfade body 150ms, URL updates |
| Sticky header | Body scrolled > 60px | Header gets bottom shadow |
| Mobile (<760px) | Viewport narrow | Becomes full-screen modal with back chevron |
```

- [ ] **Step 2: Add corresponding Figma variants for each state**

- [ ] **Step 3: Commit**

Commit message: `spec(detail-panel): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document all UI text the panel itself owns**

```markdown
## Copy

- Close button aria-label: "Close detail panel"
- Maximize button aria-label / tooltip: "Open in full view"
- Sticky-header back chevron (mobile): "Back to list"
- Doc-banner dismiss aria-label: "Dismiss this help"
- Loading state announce: "Loading details…"
- Error state title: "Couldn't load details"
- Error state body: "Something went wrong. {error.message} — Try again or close."
- Error retry button: "Retry"
```

Note: per-entity copy (titles, sublines, tab labels, action labels) is the consumer's responsibility — defined in each page's plan.

- [ ] **Step 2: Commit**

Commit message: `spec(detail-panel): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: List the FlowX DS tokens used**

```markdown
## Tokens

- Background: `color/surface/elevated`
- Border: `color/border/default`
- Header divider: `color/border/default`
- Tab active underline: `color/blue/500`
- Tab text inactive: `color/text/muted`
- Panel block bg: `color/surface/default`
- Doc banner: `color/yellow/50` bg, `color/yellow/500` left border, `color/yellow/700` text
- Type: title `font/heading/sm`, subline `font/body/sm`, tab `font/body/sm-strong`
- Spacing: header padding 14px 18px, body padding 16px 18px, panel block 12px 14px
- Shadow on maximized: `shadow/m`
- Radius: panel block 8px, action button 6px
```

- [ ] **Step 2: Commit**

Commit message: `spec(detail-panel): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Write test script**

5 tasks: (a) close panel two ways, (b) switch between tabs, (c) maximize then come back to right-slide, (d) dismiss doc banner and verify it stays dismissed on next visit, (e) interpret state matrix items by description.

- [ ] **Step 2: Recruit 5 participants — 2 ops, 2 developers, 1 support**

- [ ] **Step 3: Run sessions, capture friction points**

- [ ] **Step 4: Write summary into spec under `## User test summary`**

- [ ] **Step 5: Commit**

Commit message: `spec(detail-panel): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply test-driven changes to mockup + spec**

- [ ] **Step 2: Add `## Open questions for engineering` section**

Suggested initial items: animation library choice, URL-state persistence strategy across page navigation, accessibility focus-trap implementation, RTL support.

- [ ] **Step 3: Mark spec status as "Ready for engineering"**

- [ ] **Step 4: Commit + handoff**

Commit message: `spec(detail-panel): handoff to engineering`
