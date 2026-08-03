---
name: a11y-mobile-audit
description: Gated, platform-aware accessibility review for native mobile — detects iOS/Android/Flutter, runs a severity-ranked WCAG 2.2 AA checklist per control against the diff, proposes XCTest/Espresso/flutter_test coverage and CI wiring, covers ADA/EAA/Section 508/VPAT compliance where relevant, and writes a self-contained HTML report. Accessibility is a non-functional requirement, not a default check — use only when the user, a ticket, or an NFR explicitly asks to "audit accessibility", "run an a11y review", "check WCAG compliance", or similar on a native iOS, Android, or Flutter app. Not for implementation-time guidance (see a11y-mobile-dev), web/ARIA audits (see a11y-audit), or a default pass on every native UI PR.
license: Apache-2.0
metadata:
  authors:
    - Zoia Sharova <zoia_sharova@epam.com>
  version: "0.1.0"
---

# Accessibility Audit — Native Mobile

Full-project / per-diff accessibility review for native iOS, Android, and
Flutter: detect the platform, run a severity-ranked checklist per changed
control against WCAG 2.2 AA, propose test/CI coverage, cover regulated-app
compliance where relevant, and produce one self-contained HTML report. Pairs
with `code-review` (run by the same `tech-lead` persona) the same way the web
`a11y-audit` does — `code-review`'s Accessibility category is the
lightweight per-PR check; this skill is the full gated pass, for native.

**Not this skill's job:**
- Implementation-time technique guidance ("how do I build an accessible
  sheet on Android") — that's `../a11y-mobile-dev/SKILL.md`.
- Web/ARIA audits (React/Angular/Vue/Svelte) — that's `../a11y-audit`. A
  hybrid/WebView screen gets a11y-mobile-audit for the native chrome and
  `a11y-audit` for the embedded web content — see
  `../a11y-mobile-dev/references/patterns/webview.md`.
- End-to-end UI-automation accessibility testing beyond the platform's own
  in-process audit APIs (`XCUIApplication().performAccessibilityAudit()`,
  Espresso `AccessibilityChecks`, `flutter_test` guideline matchers) — those
  three are in scope since they're the native equivalent of unit-level
  `vitest-axe`; a full manual-tester UI-automation suite is not.

**Execution model — always gated, no mode selector.** Every finding below
gets exactly one `AskUserQuestion` and a halt until answered, then move to
the next. No Session-Mode/Review-Mode toggle system — follow the standing
rule for showing diffs and confirming before risky/config-mutating actions.

**Artifacts are project-relative**, not home-directory: reports go to
`.agents/a11y/reports/`, remediation notes to `.agents/a11y/prototypes/`, and
the append-only run log to `.agents/a11y/review-log.jsonl` — shared with the
web `a11y-audit` skill (same directory, distinguished by report filename) —
create these directories if absent.

---

## Platform detection

Run first, always. Determines which control references and testing
guidance apply. Save as `_A11Y_PLATFORM`.

```bash
_A11Y_PLATFORM="unknown"

[ -f "pubspec.yaml" ] && grep -q "flutter:" pubspec.yaml 2>/dev/null && _A11Y_PLATFORM="flutter"

if [ "$_A11Y_PLATFORM" = "unknown" ]; then
  find . -maxdepth 3 -name "*.xcodeproj" -not -path "*/Pods/*" -quit 2>/dev/null && _A11Y_PLATFORM="ios"
  find . -maxdepth 3 -name "Package.swift" -quit 2>/dev/null && _A11Y_PLATFORM="ios"
fi

if [ "$_A11Y_PLATFORM" = "unknown" ]; then
  find . -maxdepth 3 -name "build.gradle" -o -name "build.gradle.kts" 2>/dev/null | grep -q . && _A11Y_PLATFORM="android"
fi

_A11Y_UI_TOOLKIT="unknown"
if [ "$_A11Y_PLATFORM" = "ios" ]; then
  find . -maxdepth 6 -name "*.swift" -not -path "*/Pods/*" -print0 2>/dev/null | xargs -0 grep -lq "import SwiftUI" 2>/dev/null && _A11Y_UI_TOOLKIT="swiftui"
  find . -maxdepth 6 -name "*.swift" -not -path "*/Pods/*" -print0 2>/dev/null | xargs -0 grep -lq "UIViewController" 2>/dev/null && _A11Y_UI_TOOLKIT="uikit"
elif [ "$_A11Y_PLATFORM" = "android" ]; then
  find . -maxdepth 8 -name "*.kt" -print0 2>/dev/null | xargs -0 grep -lq "androidx.compose" 2>/dev/null && _A11Y_UI_TOOLKIT="compose"
  find . -maxdepth 8 -name "*.xml" -path "*/res/layout/*" -quit 2>/dev/null && _A11Y_UI_TOOLKIT="views"
fi

echo "A11Y_PLATFORM: $_A11Y_PLATFORM / A11Y_UI_TOOLKIT: $_A11Y_UI_TOOLKIT"
```

**If `_A11Y_PLATFORM` is still `unknown`**, ask via `AskUserQuestion` (iOS /
Android / Flutter) rather than guessing. A repo can be more than one — run
the audit once per platform present, using the platform-appropriate sections
of each per-control reference file.

---

## Severity model

Shared with `a11y-mobile-dev`'s AI-failure-pattern reference. Use these four
levels consistently in every finding:

- **Critical** — content or controls invisible to assistive technology
  (gesture-only handlers with no role, missing labels on icon-only controls,
  a disabled control hidden instead of dimmed, UIKit trait assignment that
  destroys `.isButton`).
- **High** — breaks under an accessibility system setting (hardcoded font
  size, hardcoded color, ignored reduce-motion/reduce-transparency,
  contrast below 4.5:1/3:1).
- **Medium** — degraded but not broken (missing hint/content description
  detail, no custom actions on a multi-action list cell, missing header
  trait on a section heading, no grouping on related content).
- **Low** — best practice (missing Voice Control/Voice Access input labels,
  no large-content/scaled-metric handling on a non-text dimension).

Finding report template (used for every finding, mirrors `a11y-audit`'s):

```
### [SEVERITY] [Short title]

**File:** `path/to/File.swift:42` (or .kt / .dart)
**WCAG:** [criterion if applicable] · **Platform guideline:** [HIG/Material/Flutter a11y guideline if applicable]
**Issue:** [1-2 sentence description]
**Assistive-tech impact:** [what VoiceOver/TalkBack/screen-reader user hears or doesn't hear]
**Fix:**
[bad code] → [corrected code]
```

---

## Step 0 — Setup

### Step 0.A — Reference manifest

Compute which `../a11y-mobile-dev/references/*.md` files apply, based on
platform and diff content. Print once, informational only — no gate.

```bash
_HAS_DIFF=$(git diff --name-only HEAD 2>/dev/null | wc -l | tr -d ' ')
[ "$_HAS_DIFF" = "0" ] && _NEW_REPO=1 || _NEW_REPO=0

_REFS_NEEDED=("wcag-mapping.md")

[ "$_A11Y_PLATFORM" = "ios" ] && _REFS_NEEDED+=("ios/ai-failure-patterns.md")
[ "$_A11Y_UI_TOOLKIT" = "swiftui" ] && _REFS_NEEDED+=("ios/swiftui-patterns.md")
[ "$_A11Y_UI_TOOLKIT" = "uikit" ] && _REFS_NEEDED+=("ios/uikit-patterns.md")

if [ "$_NEW_REPO" = "1" ]; then
  _REFS_NEEDED+=("dynamic-type.md" "color-visual.md" "motion-input.md")
else
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'font|dynamictype|textscal|sp\b' 2>/dev/null && _REFS_NEEDED+=("dynamic-type.md")
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'color|contrast|dark.?mode|theme' 2>/dev/null && _REFS_NEEDED+=("color-visual.md")
  git diff --name-only HEAD 2>/dev/null | xargs grep -qicE 'animation|motion|transition' 2>/dev/null && _REFS_NEEDED+=("motion-input.md")
fi
```

Print `LOADED` vs `NOT NEEDED FOR THIS PLATFORM` (plain text, no deletion of
any kind), then read each `NEEDED` file from `../a11y-mobile-dev/references/`
plus this skill's own `references/controls/**`, `references/notifications/**`,
and `references/patterns/**` for the changed controls (see §Audit below).

### Step 0.B — Native audit tooling check

Detect whether the platform's own in-process accessibility audit is already
wired into tests, propose adding it **in one gate** if missing — no config
file is ever created without an explicit shown diff and confirmation.

- **iOS**: check for `performAccessibilityAudit(for:` anywhere under the UI
  test target. If absent, propose the pattern from
  [references/testing.md](references/testing.md) (a `BaseAccessibilityTest`
  subclass wrapping `XCUIApplication().performAccessibilityAudit(for:)`).
- **Android**: check `build.gradle`/`build.gradle.kts` for
  `androidx.test.espresso:espresso-accessibility` or
  `AccessibilityChecks.enable()` anywhere under the instrumented test
  source set. If absent, propose adding the dependency and the one-line
  `AccessibilityChecks.enable()` call from
  [references/testing.md](references/testing.md).
- **Flutter**: check `pubspec.yaml`'s `dev_dependencies` and test files for
  `flutter_test`'s guideline matchers (`meetsGuideline(...)`). If absent,
  propose adding a `testWidgets` block using the guideline matchers from
  [references/testing.md](references/testing.md).

**One `AskUserQuestion` for the missing tooling**: **A)** add now (test file
+ dependency, shown as a diff) **B)** skip **C)** add to
`.agents/a11y/TODOS.md`. **HALT. Wait for the answer.**

### Step 0.C — Readiness summary

Print once, no gate: platform/toolkit, references loaded, native audit
tooling status (`installed` / `proposed` / `not applicable`). Never blocks.

### Step 0.D — CI system detection

Identical approach to `a11y-audit`'s Step 0.D — detect
`github-actions`/`gitlab-ci`/`azure-pipelines`/`circleci`/`jenkins`/`none`
via the presence of the matching config path/file. Determines what §Testing
& CI proposes; never assume GitHub Actions.

---

## Scope detection

If `_A11Y_PLATFORM` resolved to a known platform, scope is confirmed —
proceed to the audit for all diffs touching UI files (`.swift`, `.kt`,
`.dart` under a view/screen/widget directory). If the diff touches no UI
files and contains no accessibility-related keywords
(`accessibilit|voiceover|talkback|semantics|contentdescription|wcag`), note
"a11y-mobile-audit skipped — no UI scope detected" and stop.

---

## Audit — per-control checklist

**One finding → one `AskUserQuestion` → wait → next finding.** Do not batch.

For each control touched by the diff, read the matching file under
`references/controls/`, `references/notifications/`, or `references/patterns/`
(same filenames as `a11y-mobile-dev`'s reference set — e.g. `button.md`,
`sheet.md`, `carousel.md`) — these carry a Gherkin/acceptance-criteria
section per platform on top of the same Name/Role/Groupings/State/Focus
content `a11y-mobile-dev` uses, sourced from the Magenta QA library. Walk its
numbered test steps (keyboard actions, screen-reader gestures, screen-reader
output, device settings) against the actual diff, and flag any step that
fails as a finding using the template above.

Cross-cutting checks that apply regardless of which control changed — check
each against [../a11y-mobile-dev/references/wcag-mapping.md](../a11y-mobile-dev/references/wcag-mapping.md)'s
per-platform tables:

- Every new interactive element has a real accessible role — not a bare
  gesture handler (Critical if missing; see `ios/ai-failure-patterns.md` F1/F7).
- Every icon-only control has an accessible name (Critical if missing; F5).
- Text uses platform text styles, not hardcoded sizes (High if hardcoded; F2).
- Colors are semantic/theme-driven, not hardcoded literals (High if hardcoded; F3).
- Disabled controls are dimmed, not hidden from assistive tech (Critical if hidden; F11).
- UIKit trait changes use `.insert()`/`.remove()`, never assignment (Critical if assigned; F10).
- Animations/transitions respect the platform's reduce-motion setting (High if ignored).
- Touch targets meet the platform floor (24×24 WCAG minimum; 44×44pt iOS HIG / 48×48dp Android Material preferred).
- Modal/sheet presentation traps and returns focus correctly (Critical if focus is lost; see `../a11y-mobile-dev/references/patterns/focus.md`).

---

## Component-library / design-system audit

If the project uses a shared native design-system package (an internal
component library, or something like a company-wide SwiftUI/Compose kit),
check whether its components already satisfy the control-level contract in
`references/controls/*.md` — if a shared `PrimaryButton`/`AppSheet`-style
component is missing a required trait or label, that's a single Critical
finding at the component level rather than one per call site, with a note
that fixing the shared component fixes every consumer.

---

## Testing & CI

### Native audit API coverage

Once Step 0.B's gate is resolved, generate the test only for
platforms/toolkits present and only for new interactive components in the
diff (skip for styling-only changes). One `AskUserQuestion`: **A)** write
now **B)** skip — manual testing only **C)** add to `.agents/a11y/TODOS.md`.
See [references/testing.md](references/testing.md) for the exact
`XCUIApplication().performAccessibilityAudit(for:)` /
`AccessibilityChecks.enable()` / `flutter_test` guideline-matcher patterns
per platform, including the confirmed-public `XCUIAccessibilityAuditType`
cases (don't use unconfirmed cases like `.dynamicType`/`.missingLabel` —
they don't compile).

### Manual testing protocol

For critical flows, propose the manual screen-reader + external-keyboard +
Screen Curtain protocol from
[references/native-apps-testing.md](references/native-apps-testing.md) as a
release-checklist item — this is not something to automate, note it in the
report's TODO list instead.

### CI proposal

Using Step 0.D's detected `_CI_SYSTEM`, check whether an accessibility test
job already exists (`grep -rl "accessibilit\|a11y"` in the relevant config
path). If not, propose a job that runs the native audit test target
(XCTest/Espresso/flutter_test — not a separate e2e suite) in the matching
format (`.github/workflows/a11y-mobile.yml`, an appended `.gitlab-ci.yml`
job, etc. — same per-system mapping as `a11y-audit`). Show the complete
proposed file/snippet, `AskUserQuestion`: **A)** add now **B)** skip **C)**
add to `.agents/a11y/TODOS.md`.

---

## Compliance (regulated apps)

Only when the app is explicitly flagged as regulated (ADA/EAA/Section 508,
or the user/ticket says so) — never a default check. Read
[references/compliance.md](references/compliance.md) for VPAT documentation
requirements and what evidence a native accessibility audit needs to
produce for legal sign-off. Note this in the report as a separate,
clearly-labeled section rather than folding it into the per-control findings.

---

## HTML Report

Written once, after all findings are resolved — never mid-audit.

```bash
mkdir -p ".agents/a11y/reports"
SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "project")
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
DATETIME=$(date +%Y%m%d-%H%M%S)
REPORT_HTML=".agents/a11y/reports/${SLUG}-${BRANCH}-a11y-mobile-report-${DATETIME}.html"
```

Read `templates/report.css` (same stylesheet the web `a11y-audit` skill
uses — copied here for self-containment) and embed it verbatim in a
`<style>` tag. Sections, in order: header (project/platform/toolkit/branch/
commit/date + score), summary cards (Critical/High/Medium/Low/Already-
Passing/Tests-added/CI status/Open TODOs), audit trail (one row per control
checked, including anything skipped and why), consolidated issues table
(`#·Severity·WCAG·Platform·File:line·Issue·Assistive-tech impact·Fix
notes`), stakeholder tags (Dev/Design/QA/PM — platform-specific copy, never
generic), final TODO list (priority/WCAG/task/platform/owner/effort),
Already Passing checklist, footer (project, branch, commit, platform,
standard, generated timestamp).

After writing: print the report path and a one-line summary
(`{N} critical · {N} high · {N} TODOs`).

---

## Completion Summary, Review Log, Dashboard

**Completion summary** (printed): session findings by WCAG criterion/
severity, native audit tests added, CI status, report path, platform(s)
covered.

**Review log** — append one JSON line to `.agents/a11y/review-log.jsonl`
(shared file with `a11y-audit`, distinguished by `"skill"` field):

```bash
mkdir -p ".agents/a11y"
echo '{"skill":"a11y-mobile-audit","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","platform":"'"$_A11Y_PLATFORM"'","toolkit":"'"$_A11Y_UI_TOOLKIT"'","critical_gaps":N,"tests_added":BOOL,"ci_system":"'"$_CI_SYSTEM"'","report":"REPORT_PATH","commit":"'"$(git rev-parse --short HEAD 2>/dev/null || echo unknown)"'"}' \
  >> ".agents/a11y/review-log.jsonl"
```

**Readiness dashboard** (printed):

```
| Platform | Toolkit  | Critical Gaps | Native Tests Added | Report                 |
|----------|----------|----------------|---------------------|------------------------|
| {plat}   | {toolkit}| 0              | yes/no              | {path}                 |
```

## Rules

- Always gated — one finding, one `AskUserQuestion`, one halt. Never batch.
- Never create a new CI/test-config file without an explicit shown diff and
  confirmation; never auto-delete or uninstall anything.
- Never propose a separate e2e UI-automation accessibility suite beyond the
  platform's own in-process audit API (XCTest/Espresso/flutter_test) — that
  stays out of scope, same boundary `a11y-audit` draws for web e2e.
- Never assume GitHub Actions — always use Step 0.D's detected CI system.
- Never treat compliance (ADA/EAA/508/VPAT) as a default check — only when
  the app is explicitly flagged as regulated.
- Artifacts are project-relative under `.agents/a11y/`, never a home-dir path.
- The report is written once, after all findings are resolved — not mid-audit.
