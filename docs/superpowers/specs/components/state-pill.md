# State Pill — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The State Pill is the canonical status indicator used across the Runtime tab for every instance type — Process Instances, UI Flow Sessions, Tasks, and similar. It renders the normalized five-state vocabulary (Running, Waiting, Completed, Failed, Terminated) as a colored pill so users can scan lists without parsing engineering jargon.

On hover and on keyboard focus, the pill reveals a tooltip showing the underlying raw engineering state and a short description, so power users and operators can still reach the precise system status. The mapping from raw state to normalized state lives in the Plan 04 information model.

## Component API

## StatePill

Props: state: NormalizedStatus; rawState: string; rawDescription?: string.

Renders a colored pill with the normalized label (Running / Waiting / Completed / Failed / Terminated). On hover (and on keyboard focus for accessibility), shows tooltip:

"{rawState} · {rawDescription}"

Examples:
- state: 'failed', rawState: 'FINISHED_WITH_ERROR', rawDescription: 'raised exception during execution'
- state: 'waiting', rawState: 'ON_HOLD', rawDescription: 'paused awaiting input'

Mapping table (see Plan 04 information model) is the source of truth for which rawStates map to which normalized states and which descriptions accompany them.

## Open questions for engineering

- i18n/localization strategy for normalized labels and raw-state descriptions
- Keyboard nav details (focus visibility, Esc to dismiss the tooltip)
- Screen-reader announcements for the tooltip content
- RTL support
- Popover/tooltip library choice

## Figma + user testing — pending

- [ ] Figma frame: "StatePill / Default + Hover tooltip" for all 5 states
- [ ] User test the affordance as part of the cross-cutting docs-layer script (hover a "Failed" status pill and explain what the raw state means)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
