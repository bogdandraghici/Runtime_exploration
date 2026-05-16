# Configuration — Page Spec

> **Status:** Draft — pending Figma frames and user testing.

## Overview

Configuration merges the previously separate Active Policy and Config Param Overrides surfaces into a single two-axis page. The top axis governs **Build version** (default policy plus override table); the bottom axis governs **Config parameters** (defaults map plus override table). Both axes share an identical override grammar — target type (USER/ROLE) · target value · effective payload · priority · enabled toggle · audit — so the user learns one mental model and applies it twice.

The persistent "Test as user" resolver lives in the right column and is always visible. It resolves both axes for a given identity in one shot, returning the effective build outcome with its full resolution path and the per-param source for every config parameter, including diagnostic hints about disabled overrides that would otherwise have matched. The resolver replaces today's buried `test-effective` modal and becomes a continuous diagnostic surface.

Together, these two changes resolve friction #1 from the design spec: the shape duplication between policy and params disappears behind a shared grammar, and the question "what does this user actually get?" is answerable on the same page where the answer is configured.

## Information model

interface BuildAxis {
  defaultPolicy: { type: 'LATEST_ON_BRANCH'|'FIXED_VERSION'; branchId?: string; buildId?: string; resolvesToTag: string };
  overrides: BuildOverride[];
}

interface BuildOverride {
  id: string;
  target: { type: 'USER'|'ROLE'; value: string };
  policy: { type: 'LATEST_ON_BRANCH'|'FIXED_VERSION'; branchId?: string; buildId?: string; resolvesToTag: string };
  priority: number;
  enabled: boolean;
  audit: AuditFields;
}

interface ParamAxis {
  defaults: Record<string, ParamValue>;
  overrides: ParamOverride[];
}

interface ParamOverride {
  id: string;
  target: { type: 'USER'|'ROLE'; value: string };
  params: Record<string, ParamValue>;  // partial map — only the params this rule overrides
  priority: number;
  enabled: boolean;
  audit: AuditFields;
}

interface ResolverInput { username: string; roles: string[]; }
interface ResolverOutput {
  build: { resolvesToTag: string; source: 'default'|'override'; overrideId?: string; resolutionPath: PathStep[] };
  params: Record<string, { value: ParamValue; source: 'default'|'override'; overrideId?: string; note?: string }>;
}
type PathStep = { description: string; matched: boolean; reason?: string };

## Component API

Page props: buildAxis: BuildAxis; paramAxis: ParamAxis; resolverInput: ResolverInput; resolverOutput?: ResolverOutput; onAxisEdit; onOverrideCreate; onOverrideEdit; onOverrideToggle; onOverrideDelete; onResolve(input).

Override CRUD: opens a modal with target picker (USER/ROLE), policy/params editor, priority field, enabled toggle. Same modal component used for both axes — varies the payload editor.

Resolver: client calls `POST /runtime/configuration/resolve` with `{username, roles}`, server returns ResolverOutput. Debounced 300ms on input change.

URL: `/runtime/configuration?resolve.username=&resolve.roles=`. Resolver input pre-populates from URL so links are shareable.

## Interaction states

| State | Trigger | Visual |
|---|---|---|
| Default empty resolver | No input | "Enter a username to test" placeholder card |
| Resolver loading | onResolve in flight | Skeleton in resolved cards |
| Resolver resolved | onResolve returned | Two resolved cards (Build axis + Param axis) populated |
| Resolution path | Always under build outcome | Ordered list with matched ✓ / skipped (greyed) / not-reached states |
| Override row toggle | onOverrideToggle | Switch animates, list refreshes; resolver re-runs on debounce if input populated |
| Override delete | onOverrideDelete | Confirmation: "Delete this override? Targets {target} will fall back to {default}." |
| Override edit | Click row | Modal opens populated |
| Empty overrides | overrides.length === 0 | "No overrides yet — all callers get the default. Add an override to differentiate a user or role." |
| Disabled-but-would-match hint | param value comes from default, but a disabled override would have matched | Add italic note "{role} override would set {value} but is OFF" |

## Copy

- Page title: "Configuration"
- Help icon: "Two axes a runtime caller can be configured on — what build they run, and what params it uses."
- Doc banner: "How configuration works. Every caller has a default for each axis. Overrides match users or roles and replace the default for those callers. Higher-priority overrides win when multiple match. The resolver on the right shows the effective values for a specific user."
- Build axis title: "Build version"
- Build axis sub: "{n} overrides · {n}% on default"
- Param axis title: "Config parameters"
- Param axis sub: "{n} params · {n} with overrides"
- Default card label: "Default policy" / "Defaults"
- Override section heading: "Overrides"
- Resolver title: "Test as user"
- Resolver sub: "See exactly what a caller would get for both axes — and why."
- Resolved card titles: "Build version", "Config parameters"
- Resolved source labels: "from default", "via {USER|ROLE} override · {target} · priority {n}"
- Resolved warning hint: "{target} override would set {value} but is OFF"
- Empty overrides: "No overrides yet — all callers get the default. Add an override to differentiate a user or role."
- Empty resolver: "Enter a username to test"
- Delete confirmation: "Delete this override? Targets {target} will fall back to {default}."

## Tokens

- Axis icon — build: `color/blue/500` bg
- Axis icon — params: `color/purple/500` bg
- Default card: `color/surface/alt` bg, left border 3px matching axis color
- Target badge USER: `color/blue/50` bg, `color/blue/700` text, `color/blue/100` border
- Target badge ROLE: `color/purple/50` bg, `color/purple/500` text, `color/purple/100` border
- Toggle on: `color/green/500`; off: `color/border/strong`
- Path step matched: `color/green/700` with `color/green/500` dot + pulse
- Path step skipped: line-through, `color/text/dim`

## Open questions for engineering

- Resolver endpoint contract (extends existing test-effective).
- Preview mode ("what would change if I added X" — future).
- Param value typing (string/number/json/secret distinctions).
- Audit log retention.
- Priority collision rules.

## Figma + user testing — pending

- [ ] Figma frame: "Configuration / Default" — two axis sections stacked (Build version, Config parameters), each with default card + override table; right column resolver in empty state.
- [ ] Figma frame: "Configuration / Resolver populated" — resolver showing u_4419 with admins + beta-testers roles, both build outcome (with resolution path) and per-param resolution sources, including a "would set but OFF" hint.
- [ ] Figma frame: "Configuration / Override edit modal" — target picker, policy/params editor, priority, enabled toggle.
- [ ] Figma variants for each interaction state listed above.
- [ ] User test script — 5 tasks: (a) figure out what build u_4419 gets and why, (b) figure out what llm_temperature an admin gets, (c) create a new override for role "beta-testers" pointing to FIXED v1.4.0-beta, (d) toggle an override off and explain what happens, (e) explain the resolution path in your own words.
- [ ] Run user test with 5 participants (emphasis on developers and admins).
- [ ] Add `## User test summary` section after sessions complete.
