# Tour — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The Tour is a guided walkthrough controller that highlights the key elements of a Runtime page in sequence. The Overview tour runs automatically on a user's first visit; every other page's tour is available on demand via the footer "Tour & help" link in the sidebar.

Each tour is composed of short steps (1-2 sentences each) that overlay the page and point to a specific target via CSS selector. The Tour controller is owned by this spec; the per-page step content lives in each page's own spec so tour copy and page copy evolve together.

## Component API

## Tour

Controller props: pageId: string; steps: TourStep[]; autoRun: 'first-visit'|'never'|'always-prompt'.

interface TourStep { selector: string; title: string; body: string; placement?: 'top'|'bottom'|'left'|'right' }.

Behavior:
- Footer "Tour & help" link in sidebar always launches the current page's tour on demand.
- Overview tour: autoRun 'first-visit' for new users (stored as `tour:overview:done:{userId}`).
- Every other page: autoRun 'never'; available on demand.
- Each step is 1-2 sentences max, dismiss anytime, "Next" / "Skip tour" controls.

Initial tours (sequence of steps for each page) — copy lives in the page's own spec doc. This plan owns the controller; per-page tour content is owned by the page plan's copy task.

## Open questions for engineering

- Tour library choice
- Dismissal/completion persistence storage (localStorage vs server) for `tour:overview:done:{userId}`
- i18n/localization strategy for tour step copy
- Keyboard nav details (Next/Skip/Esc, focus trap during a step)
- RTL support
- Screen-reader announcements for step transitions

## Figma + user testing — pending

- [ ] Figma frame: "Tour / Overview / Step 1 of 4" with overlay + highlighted target
- [ ] User test the affordance as part of the cross-cutting docs-layer script (launch the Overview tour and complete it)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
