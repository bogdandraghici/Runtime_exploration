# Help Popover — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The Help Popover is a small inline `?` icon affordance that surfaces a glossary definition for a domain concept without forcing the user to leave the page. It appears next to terms-of-art across the Runtime tab — alongside labels like "Build", "Active policy", "Scorer", "Token", and similar — so any user encountering an unfamiliar concept can resolve it in place.

The popover is the lightweight entry point into the broader documentation layer: hovering reveals a 1-line definition, and a "Read more" link escalates the user into the full Glossary Panel scrolled to that term. Together with the Glossary Panel, it forms a single source-of-truth surface backed by `runtime-glossary.md`.

## Component API

## HelpPopover

Props: term: keyof Glossary; short?: string; placement?: 'top'|'bottom'|'left'|'right'.

Rendered as a small `?` icon inline. On hover/focus, opens a popover with: the glossary `short` (1-line) by default; a "Read more →" link that opens the GlossaryPanel scrolled to that term.

When clicked (not hovered), the popover pins and the popover can take keyboard focus for screen readers.

Empty popover content fallback: "No definition yet — contribute one in the runtime-glossary.md doc."

## Open questions for engineering

- i18n/localization strategy for glossary terms
- Popover library choice
- Keyboard nav details (Tab/Esc behavior, focus trap when pinned)
- RTL support
- Screen-reader announcements when the popover opens/pins

## Figma + user testing — pending

- [ ] Figma frame: "HelpPopover / Inline icon + Open popover"
- [ ] User test the affordance as part of the cross-cutting docs-layer script (find the definition of "Build" without leaving the current page)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
