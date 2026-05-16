# Runtime Glossary

> **Status:** Draft. Single source of truth for every concept in the Runtime tab.

This document is the canonical definitions reference for the Runtime tab. It is surfaced inline by HelpPopover popovers and as a side panel by GlossaryPanel across every Runtime page, so each term is defined once and consumed everywhere.

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
