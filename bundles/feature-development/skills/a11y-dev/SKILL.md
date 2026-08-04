---
name: a11y-dev
description: Gated, stack-aware accessibility review — detects the frontend stack, bootstraps missing a11y linters, runs a full WCAG 2.1+2.2 AA checklist against the diff, audits component-library risk, flags confirmed critical gaps for remediation (delegated to a11y-dev), proposes unit-test/CI coverage, and writes a self-contained HTML report. Accessibility is a non-functional requirement, not a default check — use only when the user, a ticket, or an NFR explicitly asks to "audit accessibility", "run an a11y review", "check WCAG compliance", or similar. Not for implementation-time guidance or remediation prototypes (see a11y-dev), a default pass on every UI PR, or e2e accessibility testing (see the test-automation bundle's a11y-test-automation skill).
license: Apache-2.0
metadata:
  authors:
    - Zoia Sharova <zoia_sharova@epam.com>
  version: "0.1.0"
---

# Accessibility Audit

Full-project / per-diff accessibility review: detect the stack, make sure
static a11y tooling is actually active, run the WCAG checklist, flag
component-library risk, flag confirmed critical gaps for remediation,
propose unit-test and CI coverage, and produce one
self-contained HTML report. Pairs with `code-review` (run by the same
`tech-lead` persona) — `code-review`'s Accessibility category is the
lightweight per-PR check; this skill is the full gated pass.

**Not this skill's job:**
- Implementation-time technique guidance ("how do I build an accessible
  combobox") **and remediation prototypes** — that's `../a11y-dev/SKILL.md`.
  This skill flags the confirmed gap and recommends delegating to
  `a11y-dev`; it does not generate implementation artifacts itself (see
  §5.3a).
- End-to-end accessibility testing (Playwright/Cypress + axe) — deliberately
  out of scope here; that belongs to `bundles/test-automation`'s 
  `a11y-test-automation` skill. Do not propose an e2e job or
  `@axe-core/playwright` anywhere in this flow.
- Paid tooling. Deque's axe DevTools Linter is documented in
  `references/static-analysis-tooling.md` as context only — never propose
  installing or purchasing it.

**Execution model — always gated, no mode selector.** Every finding below
gets exactly one `AskUserQuestion` and a halt until answered, then move to
the next. There is no Session-Mode/Review-Mode toggle system — you already
have a standing rule for showing diffs and confirming before risky/
config-mutating actions (installs, config patches, new files); follow that
directly instead of a separate mode subsystem.

**Artifacts are project-relative**, not home-directory: reports go to
`.agents/a11y/reports/`, and the append-only run log to
`.agents/a11y/review-log.jsonl` — create these directories if absent.
Remediation prototypes are `a11y-dev`'s artifact, not this skill's.

---

## Stack Detection

Run first, always. Determines which of `a11y-dev`'s reference files apply
and which linters/tests are relevant. Save as `_A11Y_STACK` / `_A11Y_LIB`.

```bash
_PKG=$(cat package.json 2>/dev/null || echo "{}")
_A11Y_STACK="unknown"

# Colon-anchored match — avoids '"vue"' matching '"vue-router"'/'"vuex"' etc.
echo "$_PKG" | grep -qE '"@angular/core"\s*:' && _A11Y_STACK="angular"
echo "$_PKG" | grep -qE '"vue"\s*:'            && _A11Y_STACK="vue"
echo "$_PKG" | grep -qE '"svelte"\s*:'         && _A11Y_STACK="svelte"
echo "$_PKG" | grep -qE '"react"\s*:'          && _A11Y_STACK="react"
echo "$_PKG" | grep -qE '"radix-ui"\s*:'       && _A11Y_STACK="react-shadcn"
echo "$_PKG" | grep -qE '"@radix-ui/'          && _A11Y_STACK="react-shadcn"

if [ "$_A11Y_STACK" = "unknown" ]; then
  find . -maxdepth 4 -name "*.svelte"       -not -path "*/node_modules/*" -quit 2>/dev/null && _A11Y_STACK="svelte"
  find . -maxdepth 4 -name "*.vue"          -not -path "*/node_modules/*" -quit 2>/dev/null && _A11Y_STACK="vue"
  find . -maxdepth 4 -name "*.component.ts" -not -path "*/node_modules/*" -quit 2>/dev/null && _A11Y_STACK="angular"
  find . -maxdepth 4 -name "*.tsx"          -not -path "*/node_modules/*" -quit 2>/dev/null && _A11Y_STACK="react"
  find . -maxdepth 4 -name "*.html"         -not -path "*/node_modules/*" -quit 2>/dev/null && _A11Y_STACK="html"
fi

_A11Y_LIB="none"
echo "$_PKG" | grep -qE '"@mui/material"\s*:'      && _A11Y_LIB="mui"
echo "$_PKG" | grep -qE '"@angular/material"\s*:'  && _A11Y_LIB="angular-material"
echo "$_PKG" | grep -qE '"vuetify"\s*:'            && _A11Y_LIB="vuetify"
echo "$_PKG" | grep -qE '"primeng"\s*:'            && _A11Y_LIB="primeng"
echo "$_PKG" | grep -qE '"primevue"\s*:'           && _A11Y_LIB="primevue"
echo "$_PKG" | grep -qE '"@headlessui/'            && _A11Y_LIB="headlessui"

echo "A11Y_STACK: $_A11Y_STACK / A11Y_LIB: $_A11Y_LIB"
```

**Known gap:** no Svelte UI library is fingerprinted above (e.g. Skeleton,
Svelte Material UI) — `_A11Y_LIB` will read `"none"` for a Svelte project
using one, even though §5.2 now references `svelte-a11y.md`. Until this is
added, treat `_A11Y_LIB="none"` on Svelte as inconclusive rather than "no
library in use," and ask if uncertain.

**If `_A11Y_STACK` is still `unknown`**, ask via `AskUserQuestion` (React /
React+shadcn / Angular / Vue / Svelte / Plain HTML / **None of these — not a
frontend project**) rather than guessing. If **None of these** is chosen,
set `_A11Y_STACK="none"`, note `"a11y audit skipped — no frontend stack in
this repo"` in §5.8, and stop here — do not proceed to Scope Detection,
Step 0, or any §5.x section. This is the only path in this skill that can
conclude "not applicable"; without it, a repo that merely lists a frontend
dependency somewhere (even in an unrelated workspace) has no way to be
correctly classified as out of scope.

**If `_A11Y_LIB` is `none`** on a known frontend stack, that's valid — but if
this is a *new* project with no accessible-primitives library at all, flag it
once during §5.2 (below) as the highest-risk scenario: every custom widget's
ARIA/keyboard burden is now fully manual.

---

## A11y Scope Detection

Run immediately after Stack Detection, **before** Step 0 — a diff shouldn't
trigger linter-install proposals (Step 0.B) if it turns out to have no UI
scope. This check is **not** conditional on `_A11Y_STACK` being `unknown`:
`_A11Y_STACK` is computed from `package.json`/file-glob across the **whole
repo**, so a monorepo can resolve to a known frontend stack (e.g. `react`)
even when the *current diff* is 100% backend and touches no UI at all. Skip
this check only when `_NEW_REPO=1` (no diff exists yet to filter against —
e.g. auditing a fresh checkout) or when Stack Detection already exited with
`_A11Y_STACK="none"`.

```bash
_HAS_DIFF=$(git diff --name-only HEAD 2>/dev/null | wc -l | tr -d ' ')
[ "$_HAS_DIFF" = "0" ] && _NEW_REPO=1 || _NEW_REPO=0

if [ "$_NEW_REPO" = "0" ]; then
  _UI_FILES=$(git diff --name-only HEAD 2>/dev/null | grep -cE '\.(tsx|jsx|html|vue|css|component\.ts|template\.html)$' || echo 0)
  _UI_KEYWORDS=$(git diff --name-only HEAD 2>/dev/null | xargs grep -icE 'form|modal|dialog|button|nav|menu|tab|carousel|tooltip|dropdown|input|label|focus|keyboard|screen.?reader|aria|wcag|axe|a11y|color|contrast|animation|motion' 2>/dev/null || echo 0)
fi
```

`_NEW_REPO = 1` → proceed (nothing to filter by). Otherwise: `_UI_FILES > 0`
OR `_UI_KEYWORDS > 3` → proceed to Step 0 and the audit. Both zero → note
"a11y audit skipped — no UI scope in this diff" in §5.8 and stop **here**,
before Step 0 runs.

`_NEW_REPO` is computed here (not duplicated in Step 0.A) and reused by
Step 0.A's reference-manifest logic below.

---

## Step 0 — Setup

### Step 0.A — Reference manifest

Compute which of `../a11y-dev/references/*.md` apply to this stack/diff.
Print once, informational only — no gate.

```bash
# _NEW_REPO already set by A11y Scope Detection above — reused here.
# keyboard-navigation.md and focus-management.md are always-loaded: §5.1's
# 2.4.1 (skip link) and 2.4.7 (focus visible) checks run on every diff, not
# just ones introducing a combobox/tabs/etc — gating them behind the widget
# keyword regex below would leave those checks without their backing
# reference on a plain diff.
_REFS_NEEDED=("semantic-html.md" "aria-attributes.md" "testing-a11y.md" "wcag22-new-criteria.md" "keyboard-navigation.md" "focus-management.md")

[ "$_A11Y_STACK" = "react-shadcn" ] || [ "$_A11Y_STACK" = "react" ] && _REFS_NEEDED+=("react-shadcn.md")
[ "$_A11Y_STACK" = "angular" ] && _REFS_NEEDED+=("angular-a11y.md")
[ "$_A11Y_STACK" = "vue" ]     && _REFS_NEEDED+=("vue-a11y.md")
[ "$_A11Y_STACK" = "svelte" ]  && _REFS_NEEDED+=("svelte-a11y.md")
[ "$_A11Y_LIB" != "none" ]     && _REFS_NEEDED+=("component-library-risks.md")

if [ "$_NEW_REPO" = "1" ]; then
  _REFS_NEEDED+=("mobile-touch.md" "reduced-motion.md" "forms-a11y.md" "aria-patterns.md" "keyboard-patterns.md")
else
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'touch|mobile|swipe|gesture|pointer|orientation|reflow|viewport' 2>/dev/null && _REFS_NEEDED+=("mobile-touch.md")
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'animation|transition|motion|framer|gsap' 2>/dev/null && _REFS_NEEDED+=("reduced-motion.md")
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE '<form|<input|<select|<textarea|autocomplete|fieldset' 2>/dev/null && _REFS_NEEDED+=("forms-a11y.md")
  # aria-patterns.md/keyboard-patterns.md stay gated here — they're the
  # full role/keybinding specs for complex custom widgets specifically,
  # unlike keyboard-navigation.md/focus-management.md above.
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'combobox|accordion|tabs|modal|dialog|dropdown|menu|drawer|carousel|treeview|data.?grid|toolbar' 2>/dev/null && \
    _REFS_NEEDED+=("aria-patterns.md" "keyboard-patterns.md")
fi
```

Print `LOADED` vs `NOT NEEDED FOR THIS STACK` (plain text, no `rm`/uninstall
of any kind — files are never deleted automatically), then read each
`NEEDED` file from `../a11y-dev/references/`.

### Step 0.B — Linter bootstrap

Detect missing a11y linters for the stack (see
[references/static-analysis-tooling.md](references/static-analysis-tooling.md)
for the full per-framework rationale + component-mapping guidance this
section pulls from), propose installing them **in one gate**, then patch
ESLint config only with an explicit shown diff and confirmation.

```bash
_PKG_DEV=$(node -e "const p=require('./package.json'); console.log(JSON.stringify({...(p.devDependencies||{}),...(p.dependencies||{})}))" 2>/dev/null || echo "{}")

_ESLINT_FORMAT="none"
[ -f "eslint.config.js" ] || [ -f "eslint.config.mjs" ] && _ESLINT_FORMAT="flat"
[ -f ".eslintrc.json" ] && _ESLINT_FORMAT="legacy-json"
[ -f ".eslintrc.js" ]   && _ESLINT_FORMAT="legacy-js"
[ "$_ESLINT_FORMAT" = "none" ] && echo "$_PKG" | grep -q '"eslintConfig"' && _ESLINT_FORMAT="package-json-inline"

_MISSING_LINTERS=(); _MISSING_INSTALLS=(); _MISSING_REASONS=()

if [ "$_A11Y_STACK" = "react" ] || [ "$_A11Y_STACK" = "react-shadcn" ]; then
  echo "$_PKG_DEV" | grep -qE '"eslint-plugin-jsx-a11y"\s*:' || {
    _MISSING_LINTERS+=("eslint-plugin-jsx-a11y"); _MISSING_INSTALLS+=("npm i -D eslint-plugin-jsx-a11y")
    _MISSING_REASONS+=("Catches missing alt/labels/ARIA roles at edit time")
  }
fi
[ "$_A11Y_STACK" = "angular" ] && echo "$_PKG_DEV" | grep -qE '"@angular-eslint/eslint-plugin-template"\s*:' || {
  [ "$_A11Y_STACK" = "angular" ] && {
    _MISSING_LINTERS+=("@angular-eslint/eslint-plugin-template"); _MISSING_INSTALLS+=("ng add @angular-eslint/schematics")
    _MISSING_REASONS+=("Ships the 'accessibility' template-lint preset")
  }
}
[ "$_A11Y_STACK" = "vue" ] && echo "$_PKG_DEV" | grep -qE '"eslint-plugin-vuejs-accessibility"\s*:' || {
  [ "$_A11Y_STACK" = "vue" ] && {
    _MISSING_LINTERS+=("eslint-plugin-vuejs-accessibility"); _MISSING_INSTALLS+=("npm i -D eslint-plugin-vuejs-accessibility")
    _MISSING_REASONS+=("24+ rules for Vue template a11y issues")
  }
}
[ "$_A11Y_STACK" = "svelte" ] && echo "$_PKG_DEV" | grep -qE '"eslint-plugin-svelte"\s*:' || {
  [ "$_A11Y_STACK" = "svelte" ] && {
    _MISSING_LINTERS+=("eslint-plugin-svelte"); _MISSING_INSTALLS+=("npm i -D eslint-plugin-svelte")
    _MISSING_REASONS+=("Surfaces the Svelte compiler's a11y_* warnings via svelte/valid-compile")
  }
}
[ "$_A11Y_STACK" = "html" ] && echo "$_PKG_DEV" | grep -qE '"@html-eslint/eslint-plugin"\s*:' || {
  [ "$_A11Y_STACK" = "html" ] && {
    _MISSING_LINTERS+=("@html-eslint/eslint-plugin"); _MISSING_INSTALLS+=("npm i -D @html-eslint/eslint-plugin")
    _MISSING_REASONS+=("Static a11y checks for plain HTML")
  }
}

_CSS_CHANGES=$(git diff --name-only HEAD 2>/dev/null | grep -cE '\.(css|scss|sass|less)$' || echo 0)
if [ "$_CSS_CHANGES" -gt 0 ] || [ "$_NEW_REPO" = "1" ]; then
  echo "$_PKG_DEV" | grep -qE '"stylelint-a11y"\s*:' || {
    _MISSING_LINTERS+=("stylelint-a11y"); _MISSING_INSTALLS+=("npm i -D stylelint stylelint-a11y")
    _MISSING_REASONS+=("Catches outline:none, low-contrast CSS vars, display:none on focusable elements")
  }
fi
```

**One `AskUserQuestion` for all missing linters together** (not per-linter):

```
Missing a11y linters for stack: {_A11Y_STACK}
  1. {name} → {install command} — {reason}
  2. ...

npm install only. No config file created or modified without a separate,
explicit confirmation per plugin (below).

A) Install all   B) Choose which (reply with numbers)   C) Skip all — add to TODOS.md
```

**HALT. Wait for the answer.** Run the chosen installs. For each
successfully installed plugin: if `_ESLINT_FORMAT != "none"`, show the exact
diff to activate it in that config format and use `AskUserQuestion`
(Apply now / Skip — patch manually) before touching the file. If
`_ESLINT_FORMAT = "none"`, say activation is deferred — **never create a new
ESLint config file**, and never patch an existing one without this explicit
per-plugin confirmation.

### Step 0.C — Readiness summary

Print once, no gate: stack/lib, references loaded, ESLint host/config
status, linter install results (`✅ installed` / `⚠ INSTALL FAILED — retry
manually: <command>` / `⬜ not applicable`). Never blocks.

### Step 0.D — CI system detection

Determines what §5.5 proposes — never assume GitHub Actions.

```bash
_CI_SYSTEM="none"
[ -d ".github/workflows" ]      && _CI_SYSTEM="github-actions"
[ -f ".gitlab-ci.yml" ]         && _CI_SYSTEM="gitlab-ci"
[ -f "azure-pipelines.yml" ]    && _CI_SYSTEM="azure-pipelines"
[ -d ".circleci" ]              && _CI_SYSTEM="circleci"
[ -f "Jenkinsfile" ]            && _CI_SYSTEM="jenkins"
[ -f "bitbucket-pipelines.yml" ] && _CI_SYSTEM="bitbucket-pipelines"
[ -f ".travis.yml" ]            && _CI_SYSTEM="travis"
[ -f ".buildkite/pipeline.yml" ] && _CI_SYSTEM="buildkite"
echo "CI_SYSTEM: $_CI_SYSTEM"
```

If `_CI_SYSTEM = "none"` but some *other* CI config file exists that isn't
in the list above (e.g. `drone.yml`, `.woodpecker.yml`), don't silently
treat it as "no CI" — note in §5.8 that a CI system was detected but isn't
one this skill knows how to generate a config snippet for, and skip only
the file-writing step, not the recommendation.

If `none`, note it in §5.8 and skip §5.5's file-writing step (still fine to
mention CI options in the report as a TODO).

---

## §5.1 — A11y Audit

> **Spec reference:** success criterion numbers, names, and conformance
> levels below follow the W3C **WCAG 2.2** Recommendation,
> https://www.w3.org/TR/WCAG22/. If this checklist and a reference file
> ever disagree on a level, WCAG 2.2 itself is the tiebreaker.

**One finding → one `AskUserQuestion` → wait → next finding.** Do not batch,
do not write a summary mid-audit. Check every item below against the diff;
read the cited reference before proposing a specific fix.

- **A 1.1.1** Images/icons/SVGs: meaningful `alt`, `aria-label`, or `role="img"`+`<title>`? → `semantic-html.md`
- **A 1.3.1** Semantic HTML for structure (headings, lists, tables, fieldset/legend)? → `semantic-html.md`
- **A 1.3.2** DOM order matches visual order?
- **A 1.3.3** Instructions avoid relying solely on shape/color/size/position?
- **AA 1.3.4** UI works in portrait and landscape? → `mobile-touch.md`
- **AA 1.3.5** Form inputs declare correct `autocomplete`? → `forms-a11y.md`
- **A 1.4.1** Color is not the only way to convey information?
- **AA 1.4.3** Text/background contrast ≥ 4.5:1 (3:1 large text)?
- **AA 1.4.4** Text readable at 200% zoom?
- **AA 1.4.5** Real text used instead of images of text (exception: logos)?
- **AA 1.4.10** No horizontal scroll at 320px width? → `mobile-touch.md`
- **AA 1.4.11** Non-text UI (borders, icons) ≥ 3:1 contrast?
- **AA 1.4.12** Layout survives user text-spacing overrides? → `mobile-touch.md`
- **AA 1.4.13** Hover/focus tooltips: dismissible, hoverable, persistent?
- **A 2.1.1** Every new interactive element keyboard-operable?
- **A 2.1.2** No keyboard trap?
- **A 2.1.4** Single-character keyboard shortcuts off-by-default, focus-scoped, or remappable? → `keyboard-navigation.md`
- **A 2.4.1** Skip link present? → `keyboard-navigation.md`
- **A 2.4.2** New pages/views update `document.title`?
- **A 2.4.3** Focus order follows logical reading order?
- **A 2.4.4** Link text describes destination?
- **AA 2.4.5** More than one way to find a page (nav + search, or sitemap)?
- **AA 2.4.6** Headings/labels describe their topic?
- **AA 2.4.7** Visible focus indicator on all keyboard-operable elements? → `focus-management.md`
- **AA 2.4.11** Focused elements not obscured by sticky/fixed UI? *(WCAG 2.2)* → `wcag22-new-criteria.md`
- **AA 2.5.1** Multi-touch gestures have a single-pointer alternative? → `mobile-touch.md`
- **AA 2.5.2** Activates on `pointerup`, not `pointerdown`? → `mobile-touch.md`
- **AA 2.5.3** Accessible name contains the visible label text?
- **AA 2.5.4** Motion-triggered functions (shake/tilt) have a UI alternative and can be disabled? → `mobile-touch.md`
- **AA 2.5.7** Drag interactions have a click/tap alternative? *(WCAG 2.2)* → `wcag22-new-criteria.md`
- **AA 2.5.8** Interactive targets ≥ 24×24 CSS px? *(WCAG 2.2)* → `wcag22-new-criteria.md`
- **A 3.1.1** `<html lang="...">` declared correctly?
- **A 3.2.1** Focus triggers no unexpected context change?
- **A 3.2.2** Input changes don't auto-submit/navigate without warning?
- **AA 3.2.3** Navigation consistent across pages?
- **AA 3.2.4** Same-function components labeled consistently?
- **A 3.2.6** Help mechanisms in a consistent relative position? *(WCAG 2.2)*
- **A 3.3.1** Errors detected, identified, described in text? → `forms-a11y.md`
- **A 3.3.2** Labels/instructions present for all form inputs? → `forms-a11y.md`
- **AA 3.3.3** Error-correction suggestions provided when known? → `forms-a11y.md`
- **AA 3.3.4** Legal/financial/destructive submissions reversible/confirmable?
- **A 3.3.7** Previously-entered data auto-populated in multi-step flows? *(WCAG 2.2)* → `wcag22-new-criteria.md`
- **AA 3.3.8** Auth avoids cognitive puzzles; paste + password managers work? *(WCAG 2.2)* → `wcag22-new-criteria.md`
- **A 4.1.2** New components expose name/role/value programmatically; state changes notified? → `aria-attributes.md`
- **A 4.1.3** Status messages announced via live regions without moving focus? → `aria-attributes.md`

For each finding: `{Severity}·{SC}·{Level}·{description}` — same field order
and `·` delimiter as §5.7's issues table, so findings drop straight into
report columns without reparsing. Options
at each gate: **A)** fix now **B)** skip **C)** add to `.agents/a11y/TODOS.md`.

---

## §5.2 — Component Library Audit

If `_A11Y_LIB != "none"`, load `../a11y-dev/references/component-library-risks.md`
plus the matching framework reference (`react-shadcn.md` / `angular-a11y.md` /
`vue-a11y.md` / `svelte-a11y.md` — note: rarely reached in practice for
Svelte, see the Known gap in Stack Detection) and check known risks (missing labels on MUI `TextField`/
`IconButton`, shadcn focus ring stripped by custom styling, Angular Material
form-field ARIA gaps, PrimeNg/PrimeVue components missing bound `aria-label`,
etc.). For any component flagged as inaccessible in that reference, issue as
**Critical gap** immediately — one `AskUserQuestion` per component, same
gate model as §5.1, before moving to the next.

For prop-level checks a linter can't do out of the box (see
`references/static-analysis-tooling.md`), suggest the custom-rule pattern
rather than skip the check silently.

If `_A11Y_LIB = "none"` on a confirmed frontend stack with no library imports
found in the diff, issue one **non-blocking** `AskUserQuestion` recommending
adoption of an accessible-primitives library (Radix/shadcn, Angular CDK,
Headless UI) — note the migration cost/benefit, but never block the review
on the answer.

---

## §5.3 — Interactive Widget Patterns

Load `../a11y-dev/references/aria-patterns.md` and `keyboard-patterns.md`
when the diff introduces: combobox/autocomplete, custom dropdown/menu, modal/
drawer, tabs/accordion, tree view, interactive data grid, carousel, or
toolbar. A widget with no keyboard spec is a Critical gap.

### §5.3a — Remediation recommendation (per widget with a confirmed Critical gap)

For a widget with a Critical gap where the fix involves a non-trivial
keyboard pattern (tabs, tree, combobox, modal, data grid) — skip for trivial
fixes like adding one `aria-label`, those go straight to the issues table.
This skill does not build the fix; it identifies the gap precisely enough
that `a11y-dev` can act on it without re-deriving context:

Record: `{widget name} · SC {X.Y.Z} · {stack} · {file:line} · {what's
missing — e.g. "no keyboard handler, no aria-expanded, no roving
tabindex"}`. Point to the exact spec: the widget row in `a11y-dev/SKILL.md`'s
"Building accessible interactive widgets" table, plus the matching section
of `aria-patterns.md`/`keyboard-patterns.md` loaded in §5.3.

Then `AskUserQuestion`: **A)** delegate to `a11y-dev` now (hand off widget +
SC + file:line + stack; `a11y-dev` decides whether that means a prototype,
a direct fix, or both — not this skill's call) **B)** add to
`.agents/a11y/TODOS.md` for later **C)** skip.

This keeps the **reactive** confirmation of a gap (this skill's job) fully
separate from the **proactive** "how do I build this" implementation work
(`a11y-dev`'s job) — this skill flags and hands off, it does not generate
implementation artifacts itself.

---

## §5.4 — Tooling Proposals

Re-detect installed packages (reflects Step 0.B results) before proposing —
never re-propose something already present or already explicitly skipped in
Step 0.B.

### 5.4a — Static linter status
Report per-stack linter status (installed in Step 0.B / still missing / not
applicable). Only re-propose via `AskUserQuestion` for a linter still missing
after Step 0.B (i.e. explicitly skipped or install failed there).

### 5.4b — Runtime axe overlay
- React/React-shadcn + no `@axe-core/react`: propose it — dev-only console
  warnings for dynamic ARIA state/contrast at runtime.
- Vue + no `vue-axe`: propose it — same purpose for Vue.
- **Angular: no devtools-overlay package proposed** — there is no
  widely-adopted equivalent to `@axe-core/react` for Angular. This gap is
  closed in §5.4d instead: Angular gets automated per-component a11y
  coverage via unit tests (`axe-core` against `TestBed` fixtures, or
  `jest-axe` on Jest-based Angular projects) rather than a runtime overlay.
- **Svelte: no devtools-overlay package proposed** — no dedicated package
  exists (see `svelte-a11y.md`); closed in §5.4d via `vitest-axe`.
- **Plain HTML: no devtools-overlay package proposed** — no build step to
  hook a dev-only overlay into.

### 5.4c — Storybook addon
If Storybook is detected and `@storybook/addon-a11y` isn't: propose it — every
story gets an automatic axe pass, works across all frameworks Storybook
supports.

### 5.4d — Unit test generation

Driven by whichever test framework Step 0 detected:

| Stack | Test approach |
|---|---|
| React / Vue (Vitest) | `vitest-axe`: `expect(await axe(container)).toHaveNoViolations()` |
| React / Vue (Jest) | `jest-axe`, same assertion shape |
| Angular (Jasmine/Karma) | Run `axe-core` directly against `fixture.nativeElement` after `TestBed` renders the component |
| Angular (Jest-based) | `jest-axe` against the rendered fixture, same as React |
| Svelte (Vitest) | `vitest-axe` against the rendered component |

Run only when a test framework is detected **and** new interactive
components exist in the diff — skip for CSS-only/styling-only changes.
`AskUserQuestion` once: **A)** write tests now **B)** skip — manual testing
only **C)** add to `.agents/a11y/TODOS.md` for a follow-up PR. If **A**,
install the axe test library if missing, then ask placement (colocated vs.
`__tests__` vs. a specified top-level test folder) before writing — one
gate, not per-file.

Generate one test file per interactive component with `it()`s per §5.1
finding for that component (missing `aria-label` → expect violation without,
none with; missing `alt` → same shape; unlabeled form input → same shape).
Findings axe can't detect at the unit level (contrast, CSS focus ring) get no
test case — they stay tracked in the issues table and TODO list.

**No e2e test proposal** — static analysis + unit tests only, per this
skill's scope (e2e is `a11y-test-automation`'s future job).

Append a short "Accessibility Testing" subsection to whatever test-plan
artifact `code-review`/`implement-feature` already produces for this PR, if
one exists — otherwise this becomes part of the §5.7 report only.

---

## §5.5 — CI Proposal

Using Step 0.D's detected `_CI_SYSTEM` check whether an a11y check already exists in that system's config
(`grep -rl "a11y\|axe\|accesslint"` in the relevant config path). If not, and
`_CI_SYSTEM != "none"`, propose a **static-analysis-only** check (lint step
+ unit-test run — no e2e job, per scope) in the matching format:

- `github-actions` → `.github/workflows/a11y.yml`
- `gitlab-ci` → an `a11y` job appended to `.gitlab-ci.yml`
- `azure-pipelines` → a stage appended to `azure-pipelines.yml`
- `circleci` → a job appended to `.circleci/config.yml`
- `jenkins` → a stage appended to the `Jenkinsfile`
- `bitbucket-pipelines` / `travis` / `buildkite`: detected but no
  file-snippet generator yet — skip file-writing, note in §5.8/report that
  a11y CI is recommended but must be added manually for this CI system.

Show the **complete** proposed file/snippet, then `AskUserQuestion`:
**A)** add now **B)** skip **C)** add to `.agents/a11y/TODOS.md`. If `none`
detected, skip file-writing and just note the recommendation in the report.

---

## §5.6 — Failure Modes

Check each new UI codepath for these silent-failure patterns; flag as
**Critical** when no test/framework built-in covers it and the failure would
be invisible to assistive tech. **No per-item `AskUserQuestion` gate here**
— unlike §5.1, these get recorded directly into the issues table/TODO list
and surfaced in §5.7's report, not halted on one at a time.

- Modal opens but focus isn't moved in
- Form error rendered visually with no `aria-live`/`role="alert"`
- Icon-only button with no accessible name
- Sticky header added with no `scroll-margin-top` fix (obscures focus, 2.4.11)
- Drag-and-drop with no keyboard reorder alternative
- Route change with no title update/announcement
- Custom `<div>` used as an interactive element without role/tabindex/key handlers
- `aria-hidden="true"` on an ancestor of a focusable element

---

## §5.7 — HTML Report

Written once, after all findings from §5.1–§5.6 are resolved — never
mid-audit.

```bash
mkdir -p ".agents/a11y/reports"
SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "project")
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
DATETIME=$(date +%Y%m%d-%H%M%S)
REPORT_HTML=".agents/a11y/reports/${SLUG}-${BRANCH}-a11y-report-${DATETIME}.html"
```

Read `templates/report.css` and embed it verbatim in a `<style>` tag — don't
summarize or invent new classes; the file defines the full set used below.
If `templates/report.css` is missing, don't silently fall back to inventing
CSS: say so, halt, and ask whether to proceed with a minimal unstyled report
or locate the file — inventing classes would produce a report inconsistent
with every other report this skill has written.
Sections, in order: header (project/stack/branch/commit/date + score),
summary cards (Critical/Warnings/Already-Passing/Tests-exist/Tooling/Open
TODOs), audit trail (one row per step that ran, including anything skipped
and why), consolidated issues table (`#·Severity·SC·Level·Source·File:line·
Issue·Reasoning·Fix notes`, severity then SC order), stakeholder tags
(Dev/Design/QA/PM, section-specific copy — never generic), inline SVG
diagrams for contrast/focus-obscured/target-size/DOM-order issues (CSS-var
colors only, `role="img"` + `<title>`), final TODO list (priority/SC/task/
source/owner/effort), Already Passing checklist, footer (project, branch,
commit, stack, standard, generated timestamp).

After writing: print the report path and a one-line summary
(`{N} critical · {N} warnings · {N} TODOs`).

---

## §5.8 — Completion Summary, Review Log, Dashboard

**Completion summary** (printed): session findings by SC/severity, linters
added, tests written, CI status, report path, a11y scope flag.

**Review log** — append one JSON line:

```bash
mkdir -p ".agents/a11y"
echo '{"skill":"a11y-audit","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","stack":"'"$_A11Y_STACK"'","lib":"'"$_A11Y_LIB"'","a11y_scope":BOOL,"critical_gaps":N,"linters_added":["LIST"],"ci_system":"'"$_CI_SYSTEM"'","report":"REPORT_PATH","commit":"'"$(git rev-parse --short HEAD 2>/dev/null || echo unknown)"'"}' \
  >> ".agents/a11y/review-log.jsonl"
```

**Readiness dashboard** (printed):

```
| A11y Scope | Stack    | Critical Gaps | Linters Added | Report                 |
|------------|----------|---------------|---------------|------------------------|
| YES/NO     | {stack}  | 0             | {list}        | {path}                 |
```

## Rules

- Always gated — one **audit finding** (§5.1, §5.2's critical-gap flags,
  §5.3a's per-widget recommendation), one `AskUserQuestion`, one halt. Never
  batch findings. Two explicit exceptions, not violations of this rule:
  §5.6's failure modes are recorded directly into the issues table/report
  without a per-item gate (see §5.6); and setup/tooling steps designed as
  one gate for multiple items — Step 0.B's linter install, §5.4a's linter
  status, and §5.5's CI proposal each use a single combined gate on purpose.
- Never create a new ESLint/CI config file without an explicit shown diff
  and confirmation; never auto-delete or uninstall anything.
- Never propose e2e tests, `@axe-core/playwright`, or any paid tool.
- Never assume GitHub Actions — always use Step 0.D's detected CI system.
- Artifacts are project-relative under `.agents/a11y/`, never a home-dir path.
- The report is written once, after all findings are resolved — not mid-audit.
