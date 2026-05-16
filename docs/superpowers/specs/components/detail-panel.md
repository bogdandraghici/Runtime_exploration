# Detail Panel — Component Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

The detail panel is the single reusable right-slide surface that every entity detail view in the Runtime tab consumes. It is the foundation piece of the Runtime redesign: Instances, Incidents, Builds, Trigger, and Task detail views all render through this one component, with consumer pages supplying the data and tab content.

Architecturally it is one component with a strict API. The panel composes a header (crumb, typed ID, title, subline, status pill, meta chips, primary actions), an optional dismissible doc banner, a tabbed body, and `.panel` content blocks. The default mode is a 460–480px right-slide; it can be maximized to a full route and closed back to URL state.

By centralizing this layout, we resolve the long-standing friction of inconsistent detail surfaces across Runtime and give each consumer page a predictable, composable shell to plug into.

## Information model

The panel consumes a single typed data object. Consumers assemble this and pass it through props; the panel owns layout and chrome, not content semantics.

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

## Component API

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

## Copy

- Close button aria-label: "Close detail panel"
- Maximize button aria-label / tooltip: "Open in full view"
- Sticky-header back chevron (mobile): "Back to list"
- Doc-banner dismiss aria-label: "Dismiss this help"
- Loading state announce: "Loading details…"
- Error state title: "Couldn't load details"
- Error state body: "Something went wrong. {error.message} — Try again or close."
- Error retry button: "Retry"

Note: per-entity copy (titles, sublines, tab labels, action labels) is owned by the consumer page.

## Tokens

FlowX DS tokens:

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

## Open questions for engineering

- Animation library choice
- URL-state persistence strategy across page navigation
- Accessibility focus-trap implementation
- RTL support

## Figma + user testing — pending

- TODO: Build Figma frames (default, loading, error, empty tab, closed, maximized, tab-switch, sticky header, mobile).
- TODO: Run user test with 5 participants (2 ops, 2 dev, 1 support) per Plan 01 Task 7 script.
