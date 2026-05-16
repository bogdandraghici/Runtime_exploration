# Glossary Panel — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The Glossary Panel is a right-slide panel that lists every term defined in `runtime-glossary.md`. It is the full-text destination behind every Help Popover's "Read more" link, and is also reachable from a "?" button in the sidebar footer and from a global Cmd+K (or `?`) hotkey anywhere in the Runtime tab.

The panel reuses the chrome from the Plan 01 detail-panel for visual consistency and includes a live-filter search box at the top. Because it reads the same glossary file that powers the Help Popover, both surfaces share a single source of truth.

## Component API

## GlossaryPanel

Right-slide panel (same chrome as Plan 01 detail-panel for visual consistency) listing every term in `runtime-glossary.md`. Reachable from:
- Any HelpPopover's "Read more →" link (scrolls to specific term)
- A "?" button in the sidebar footer (opens panel at top)
- Cmd+K (or `?` keyboard shortcut) global hotkey

Props: open: boolean; scrollToTerm?: string; onClose.

Implementation: panel reads the same glossary file used by HelpPopover — single source of truth.

Search box at the top filters terms in real-time.

## Open questions for engineering

- i18n/localization strategy for glossary terms
- Keyboard nav details (Cmd+K vs `?` conflict resolution, Tab order through terms, Esc to close, focus trap)
- RTL support
- Screen-reader announcements when the panel opens and when a term is scrolled into view
- Popover library choice (for any inline affordances inside the panel)

## Figma + user testing — pending

- [ ] Figma frame: "GlossaryPanel / Open" showing term list with one term highlighted as the scroll target
- [ ] User test the affordance as part of the cross-cutting docs-layer script (open the glossary, search for "scorer", read it)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
