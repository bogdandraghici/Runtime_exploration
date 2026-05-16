# Empty State — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The Empty State is a pattern enforced across every list, panel, and tab in the Runtime tab. Whenever a surface has no data to display, an Empty State replaces it with a short explanation of (a) what would appear in this surface, and (b) how it gets there — turning the absence of data into a teaching moment instead of a dead end.

Empty States come in two tones: `neutral` when an absence is merely expected (no instances yet), and `positive` when an absence is good news (no open incidents). Canonical copy is fixed per page to ensure consistency across the Runtime tab.

## Component API

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

## Open questions for engineering

- i18n/localization strategy for canonical empty-state copy
- Screen-reader announcements when an empty state replaces a loading state
- Keyboard nav details (focus management on the CTA)
- RTL support

## Figma + user testing — pending

- [ ] Figma frames for neutral and positive tones
- [ ] User test the affordance as part of the cross-cutting docs-layer script (interpret an empty state on the Tasks page and explain how a task gets there)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
