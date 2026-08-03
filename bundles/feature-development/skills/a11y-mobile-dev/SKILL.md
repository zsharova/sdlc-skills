---
name: a11y-mobile-dev
description: Implement accessible native mobile UI — iOS (SwiftUI/UIKit), Android (Views/Compose), and Flutter — VoiceOver/TalkBack labels and traits, Dynamic Type/font-scale parity, focus and grouping, touch target size, and per-control accessibility patterns (button, dropdown, sheet, carousel, data row, and more). Use when building or reviewing a native screen/component; when asked to "make this accessible" / "add VoiceOver support" / "fix a11y" on iOS, Android, or Flutter; or before authoring any custom interactive control on a native platform. Not for web/ARIA patterns (see a11y-dev), auditing existing native UI, running a WCAG checklist, or generating a report — see a11y-mobile-audit.
license: Apache-2.0
metadata:
  authors:
    - Zoia Sharova <zoia_sharova@epam.com>
  version: "0.1.0"
---

# Accessibility — Native Mobile Implementation

This skill is **implementation-time** technique guidance for native iOS,
Android, and Flutter — control-by-control accessibility patterns (name,
role, state, focus, grouping), platform text-scaling and color/motion
system-preference handling, and the 11 documented AI-generated-code failure
patterns for iOS. It is not a review, checklist, or reporting skill. For a
full gated audit, per-platform stack detection, severity-ranked findings,
testing/CI proposals, or an HTML report, use the sibling skill
**`a11y-mobile-audit`** (paired with `code-review`, run by `tech-lead`). This
skill only answers "how do I build this correctly" — `a11y-mobile-audit`
answers "does what already exists comply, and what's missing."

For **web** UI (React/Angular/Vue/Svelte, ARIA roles, HTML semantics), use
the sibling skill `a11y-dev` instead — the two don't overlap except at a
WebView boundary (see [references/patterns/webview.md](references/patterns/webview.md)).

## When NOT to use this

- **A hybrid/WebView screen** — that's rendering HTML; use `a11y-dev` for
  the content inside the WebView and this skill only for the native chrome
  around it. See [references/patterns/webview.md](references/patterns/webview.md).
- **Internal developer tooling with a single known user** — the same
  reasoning `a11y-dev` uses: invest proportionally to how broadly the app is
  distributed.
- **A compliance issue in a third-party SDK/dependency you can't modify** —
  document it, raise it with the vendor; don't patch code you don't own.
- **A purely static, non-interactive screen** — semantic labeling is still
  the right default, but a full custom-control accessibility pass adds
  complexity with no user benefit here.

## Core concepts (cross-platform)

### 1. Name, role, state, value — one announcement
Every control needs a **name** (what it is), **role** (what kind of control —
button, link, toggle), **state** (selected/disabled/expanded), and, for
adjustable controls, a **value** — all read in a single announcement when
focus lands on it. This is the cross-platform equivalent of ARIA's
name/role/value model. Standard controls (`Button`, `UIButton`, Android
`Button`/Compose `Button`, Flutter `ElevatedButton`) get this for free;
custom controls built from a bare tappable view do not — see
[references/ios/ai-failure-patterns.md](references/ios/ai-failure-patterns.md)
(F1, F7) for the exact failure this produces on iOS, and
[references/controls/](references/controls/) for the per-control contract
across all three platforms.

### 2. Grouping — one swipe stop, not several
Related content (an icon + label, a price + product name, a list-cell's
title + subtitle) should read as one announcement, not a sequence of
separate stops. iOS: `.accessibilityElement(children: .combine)` / `.ignore`
+ custom label (SwiftUI), `shouldGroupAccessibilityChildren` (UIKit).
Android: `Modifier.semantics(mergeDescendants = true)` (Compose),
`android:screenReaderFocusable` on a `ViewGroup` (Views). Flutter:
`MergeSemantics`. Never group a container that *also* gets an interactive
trait (`.isButton`/`clickable`) via `.combine` — that causes double
activation; use `.ignore` + a custom label instead. See
[references/ios/swiftui-patterns.md](references/ios/swiftui-patterns.md) and
[references/ios/uikit-patterns.md](references/ios/uikit-patterns.md).

### 3. Focus management
Manage focus the same way you would on web: move focus into a newly
presented sheet/modal, return it to the trigger on dismiss, never let focus
land nowhere. iOS: `accessibilityViewIsModal` (must be a direct child of the
container whose siblings should be hidden) + `accessibilityPerformEscape`
for the two-finger-Z dismiss gesture (UIKit); `.sheet`/`.alert` handle this
automatically in SwiftUI. See
[references/patterns/focus.md](references/patterns/focus.md) and
[references/notifications/modal.md](references/notifications/modal.md) for
the full pattern across all three platforms.

### 4. Text scaling, color/contrast, and motion
Every platform has a font-scale system setting (iOS Dynamic Type, Android
`Configuration.fontScale`, Flutter `MediaQuery.textScaler`) and a
reduce-motion setting (iOS `accessibilityReduceMotion`, Android
`ANIMATOR_DURATION_SCALE`, Flutter `MediaQuery.disableAnimations`) — respect
both, and never hardcode text size or convey state through color alone. See
[references/dynamic-type.md](references/dynamic-type.md),
[references/color-visual.md](references/color-visual.md), and
[references/motion-input.md](references/motion-input.md) for the
full iOS-deep treatment plus Android/Flutter parallels, and
[references/patterns/animation.md](references/patterns/animation.md) for the
Magenta-sourced cross-platform version.

### 5. Touch target size
Minimum 24×24pt/dp (WCAG 2.5.8 AA floor) — but prefer the platform-native
recommendation where it's larger: 44×44pt on iOS (Apple HIG), 48×48dp on
Android (Material). Never rely on a visually large tap area backed by a
programmatically small accessible frame — set `accessibilityActivationPoint`/
`accessibilityPath` (iOS) or an explicit `minWidth`/`minHeight` if the visual
and hit-test bounds diverge.

## The #1 native AI failure: gesture handlers instead of real controls

`onTapGesture` (SwiftUI), a bare `Modifier.clickable` without semantics
(Compose), or a raw `GestureDetector` (Flutter) on a non-semantic widget all
share the same failure: they add a tap handler but **no accessibility
role**. VoiceOver/TalkBack skip the element entirely; Switch Control/Switch
Access can't scan to it; Voice Control/Voice Access can't target it. Always
reach for the platform's real control (`Button`, `UIButton`,
Compose `Button`, Flutter `ElevatedButton`/`GestureDetector` wrapped in
`Semantics(button: true)`) first. Full pattern, plus 10 more documented iOS
AI-failure modes with fix pairs (hardcoded fonts, hardcoded colors, missing
image-button labels, trait assignment instead of insertion, hidden disabled
controls, and more): [references/ios/ai-failure-patterns.md](references/ios/ai-failure-patterns.md).

## Building accessible controls

Ask "how do I build an accessible dropdown/sheet/carousel on iOS/Android/
Flutter" **before** writing the component. Each reference below documents
the Name/Role/Groupings/State/Focus contract with parallel iOS, Android, and
Flutter implementation notes and example announcement text.

| Category | Reference |
|---|---|
| Button, link, chip, toggle switch, checkbox, radio button | [references/controls/button.md](references/controls/button.md), [link.md](references/controls/link.md), [chip.md](references/controls/chip.md), [toggle-switch.md](references/controls/toggle-switch.md), [checkbox.md](references/controls/checkbox.md), [radio-button.md](references/controls/radio-button.md) |
| Text input, search | [references/controls/text-input.md](references/controls/text-input.md), [search.md](references/controls/search.md) |
| Dropdown, menu, sidebar menu | [references/controls/dropdown.md](references/controls/dropdown.md), [menu.md](references/controls/menu.md), [sidebar-menu.md](references/controls/sidebar-menu.md) |
| Sheet, expandable/disclosure | [references/controls/sheet.md](references/controls/sheet.md), [expandable.md](references/controls/expandable.md) |
| Carousel, pagination control | [references/controls/carousel.md](references/controls/carousel.md), [pagination-control.md](references/controls/pagination-control.md) |
| Slider, stepper, segmented control | [references/controls/slider.md](references/controls/slider.md), [stepper.md](references/controls/stepper.md), [segmented-control.md](references/controls/segmented-control.md) |
| Calendar/date picker, time picker, date-time picker | [references/controls/calendar-date-picker.md](references/controls/calendar-date-picker.md), [time-picker.md](references/controls/time-picker.md), [date-time-picker.md](references/controls/date-time-picker.md) |
| Progress indicator, step indicator, timer | [references/controls/progress-indicator.md](references/controls/progress-indicator.md), [step-indicator.md](references/controls/step-indicator.md), [timer.md](references/controls/timer.md) |
| Table row button, reorder data row, captcha | [references/controls/table-row-button.md](references/controls/table-row-button.md), [reorder-data-row.md](references/controls/reorder-data-row.md), [captcha.md](references/controls/captcha.md) |
| Alert dialog, modal, snackbar/toast | [references/notifications/alert-dialog.md](references/notifications/alert-dialog.md), [modal.md](references/notifications/modal.md), [snackbar-toast.md](references/notifications/snackbar-toast.md) |
| Animation, field errors, focus, headings | [references/patterns/animation.md](references/patterns/animation.md), [field-errors.md](references/patterns/field-errors.md), [focus.md](references/patterns/focus.md), [headings.md](references/patterns/headings.md) |
| Graphics/visual elements, decorative images, loading icon/spinner | [references/patterns/graphics-visual-elements.md](references/patterns/graphics-visual-elements.md), [image-decorative.md](references/patterns/image-decorative.md), [loading-icon.md](references/patterns/loading-icon.md), [loading-spinner.md](references/patterns/loading-spinner.md) |
| Strike-through text, table, tidbit, webview | [references/patterns/strike-through.md](references/patterns/strike-through.md), [table.md](references/patterns/table.md), [tidbit.md](references/patterns/tidbit.md), [webview.md](references/patterns/webview.md) |

## Per-platform deep references

| Platform | Reference |
|---|---|
| iOS — AI failure patterns (11 modes, ❌/✅ pairs) | [references/ios/ai-failure-patterns.md](references/ios/ai-failure-patterns.md) |
| iOS — SwiftUI modifiers | [references/ios/swiftui-patterns.md](references/ios/swiftui-patterns.md) |
| iOS — UIKit elements/containers/traits | [references/ios/uikit-patterns.md](references/ios/uikit-patterns.md) |
| iOS/Android/Flutter — Dynamic Type / text scaling | [references/dynamic-type.md](references/dynamic-type.md) |
| iOS/Android/Flutter — color, contrast, dark mode | [references/color-visual.md](references/color-visual.md) |
| iOS/Android/Flutter — motion, Switch Control/Access, Voice Control/Access | [references/motion-input.md](references/motion-input.md) |
| iOS 17/18/26 — new accessibility APIs | [references/ios/new-features.md](references/ios/new-features.md) |
| iOS/Android/Flutter — WCAG 2.2 AA → platform API mapping | [references/wcag-mapping.md](references/wcag-mapping.md) |

Android and Flutter don't yet have dedicated deep-dive files the way iOS
does (no `android-*`/`flutter-*` equivalents to `ios/swiftui-patterns.md`) —
the per-control references above carry Android/Flutter implementation notes
directly. Add Android/Flutter deep-dives here if a dedicated source (like
the EPAM iOS skill) becomes available for those platforms.

## Decision tree

| Situation | Where to look |
|---|---|
| Building a specific control/pattern | Component table above |
| Writing SwiftUI/UIKit-specific accessibility code | Per-platform deep references above |
| Suspect your generated code has one of the classic AI failure patterns | [references/ios/ai-failure-patterns.md](references/ios/ai-failure-patterns.md) |
| Need the WCAG criterion ↔ platform API mapping | [references/wcag-mapping.md](references/wcag-mapping.md) |
| Doing a pre-merge accessibility pass, need a full gated audit or a report | `a11y-mobile-audit` (sibling skill, run by `tech-lead`) |
| The screen is a WebView/hybrid | [references/patterns/webview.md](references/patterns/webview.md), then `a11y-dev` for the web content |

## Rules

- Never use a gesture-only handler (`onTapGesture`, bare `clickable`,
  ungrouped `GestureDetector`) on an interactive element — always use the
  platform's real control or explicitly add its semantics/traits.
- Never ship an icon-only control without an accessible name.
- Never assign accessibility traits by replacement on iOS UIKit
  (`accessibilityTraits = .selected`) — always `.insert()`/`.remove()`, or
  the existing traits (like `.isButton`) are destroyed.
- Never hide a disabled control from assistive technology — disable it
  (`.disabled()`/`enabled = false`) so it's still discoverable as "dimmed."
- Never hardcode font size or convey state through color alone on any
  platform.
- Never remove all motion for `reduceMotion`/animator-scale-zero users —
  replace it with a crossfade, not silence.
- When building a custom interactive control, implement the full
  name/role/state/value contract from the relevant `references/controls/`
  file — a partial implementation is a common source of Critical gaps at
  review time.
- If a non-trivial custom control is being hand-rolled with no
  accessible-primitives equivalent available, raise it with `tech-lead`
  rather than silently shipping.
