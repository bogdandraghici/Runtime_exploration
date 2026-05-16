# Configuration Page — Plan 07

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax.

**Goal:** Specify the merged Configuration page — Active Policy + Config Param Overrides on one surface with a persistent "Test as user" resolver. Resolves friction #1.

**Architecture:** Two-axis page (Build version + Config parameters), shared override grammar across both axes, right-side persistent resolver panel that returns both outcomes for a given identity with full resolution path.

**Deliverables:**
- Create: `docs/superpowers/specs/pages/configuration.md`
- Create: Figma frames "Configuration / Default", "… / Resolver populated", "… / Override edit modal"

**Depends on:** Plan 02.

---

### Task 1: Define information model

- [ ] **Step 1: Document the unified override model**

```markdown
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
```

- [ ] **Step 2: Commit**: `spec: configuration information model`

---

### Task 2: Define component API

- [ ] **Step 1: Document props, events, and resolver API**

```markdown
## Component API

Page props: buildAxis: BuildAxis; paramAxis: ParamAxis; resolverInput: ResolverInput; resolverOutput?: ResolverOutput; onAxisEdit; onOverrideCreate; onOverrideEdit; onOverrideToggle; onOverrideDelete; onResolve(input).

Override CRUD: opens a modal with target picker (USER/ROLE), policy/params editor, priority field, enabled toggle. Same modal component used for both axes — varies the payload editor.

Resolver: client calls `POST /runtime/configuration/resolve` with `{username, roles}`, server returns ResolverOutput. Debounced 300ms on input change.

URL: `/runtime/configuration?resolve.username=&resolve.roles=`. Resolver input pre-populates from URL so links are shareable.
```

- [ ] **Step 2: Commit**: `spec: configuration component API`

---

### Task 3: Build hi-fi static mockup

- [ ] **Step 1: Default mockup**

Figma "Configuration / Default": two axis sections stacked (Build version, Config parameters), each with default card + override table. Right column resolver in empty state ("Enter a username to test").

- [ ] **Step 2: Resolver populated**

Frame: resolver showing u_4419 with admins + beta-testers roles, showing both build outcome (with resolution path) and per-param resolution sources (including a "would set but OFF" hint).

- [ ] **Step 3: Override edit modal**

Frame: modal for creating/editing an override — target picker, policy/params editor, priority, enabled.

- [ ] **Step 4: Commit**: `spec(configuration): default + resolver + modal mockups`

---

### Task 4: Define interaction states

- [ ] **Step 1: Document state matrix**

```markdown
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
```

- [ ] **Step 2: Add Figma variants**

- [ ] **Step 3: Commit**: `spec(configuration): interaction states`

---

### Task 5: Specify copy

- [ ] **Step 1: Document copy**

```markdown
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
```

- [ ] **Step 2: Commit**: `spec(configuration): copy`

---

### Task 6: Identify design tokens

- [ ] **Step 1: Token list**

```markdown
## Tokens

- Axis icon — build: `color/blue/500` bg
- Axis icon — params: `color/purple/500` bg
- Default card: `color/surface/alt` bg, left border 3px matching axis color
- Target badge USER: `color/blue/50` bg, `color/blue/700` text, `color/blue/100` border
- Target badge ROLE: `color/purple/50` bg, `color/purple/500` text, `color/purple/100` border
- Toggle on: `color/green/500`; off: `color/border/strong`
- Path step matched: `color/green/700` with `color/green/500` dot + pulse
- Path step skipped: line-through, `color/text/dim`
```

- [ ] **Step 2: Commit**: `spec(configuration): design tokens`

---

### Task 7: User test

- [ ] **Step 1: Script**

5 tasks: (a) figure out what build u_4419 gets and why, (b) figure out what llm_temperature an admin gets, (c) create a new override for role "beta-testers" pointing to FIXED v1.4.0-beta, (d) toggle an override off and explain what happens, (e) explain the resolution path in your own words.

- [ ] **Step 2: Run with 5 participants (emphasis on developers and admins)**

- [ ] **Step 3: `## User test summary`**

- [ ] **Step 4: Commit**: `spec(configuration): user-test summary`

---

### Task 8: Iterate + handoff

- [ ] **Step 1: Apply changes**

- [ ] **Step 2: `## Open questions for engineering`**

Items: resolver endpoint contract (extends existing test-effective), preview mode ("what would change if I added X" — future), param value typing (string/number/json/secret distinctions), audit log retention, priority collision rules.

- [ ] **Step 3: Mark "Ready for engineering"**

- [ ] **Step 4: Commit**: `spec(configuration): handoff`
