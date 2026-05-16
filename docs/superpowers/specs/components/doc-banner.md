# Doc Banner — Component Spec

> **Status:** Draft — pending Figma frame and user testing.

## Overview

The Doc Banner is a contextual, dismissible help strip that appears above panel content or inside a tab body across the Runtime tab. It carries page-level or section-level guidance — what this surface is for, the one or two things a user should know before acting — without occupying permanent screen real estate.

Banners are yellow by default (severity `info`) and orange for the `warn` variant. Dismissal is sticky per user, but rebroadcasts when the underlying copy changes via a content hash, so meaningful updates re-surface even to users who dismissed the previous version.

## Component API

## DocBanner

Props: id: string (stable across versions); title: string; body: string; dismissible?: boolean = true; severity?: 'info'|'warn' = 'info'.

Dismissal: when user clicks ×, persist `dismissed:{id}:{userId}:{contentHash}`. Re-show only when contentHash changes (i.e. when the banner copy meaningfully changes).

Placement: above panel content or inside a tab body. The yellow severity is the default; warn variant uses orange.

Accessibility: `role="region" aria-label={title}"`; dismiss button has `aria-label="Dismiss this help"`.

## Open questions for engineering

- Dismissal persistence storage (localStorage vs server)
- i18n/localization strategy for banner copy and the resulting contentHash
- Screen-reader announcements when a banner appears
- Keyboard nav details (focus order, Esc to dismiss?)
- RTL support

## Figma + user testing — pending

- [ ] Figma frame: "DocBanner / Default · Dismissed"
- [ ] User test the affordance as part of the cross-cutting docs-layer script (dismiss a doc banner and verify it stays dismissed; navigate from a doc banner's "Read more" link to the glossary)
- [ ] Capture findings and add `## User test summary` section
- [ ] Mark "Ready for engineering" once Figma + user testing complete
