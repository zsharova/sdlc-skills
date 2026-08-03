# Motion, Animation & Alternative Input — iOS, Android, Flutter

Supports `a11y-mobile-dev`. Covers reduce motion, animation safety, Switch
Control, Voice Control, and Full Keyboard Access. The iOS section is the deep
reference; Android and Flutter get a shorter parallel section.

## iOS — Reduce Motion

```swift
// Bad — ignores user preference
withAnimation(.spring(response: 0.5, dampingFraction: 0.3)) {
    showDetail.toggle()
}

// Good — respects reduce motion
@Environment(\.accessibilityReduceMotion) var reduceMotion
withAnimation(reduceMotion ? .none : .spring()) { showDetail.toggle() }

// Good — replace motion with crossfade, not removal
.transition(reduceMotion ? .opacity : .slide)
.animation(reduceMotion ? .easeInOut(duration: 0.2) : .spring(), value: state)
```

Key principle: don't remove *all* animation when reduce motion is enabled —
replace aggressive motion with a subtle crossfade/opacity transition.
Complete removal feels broken, not accessible.

**Needs a reduce-motion alternative:** parallax effects, scaling/zooming
transitions, sliding/pushing transitions, spinning/rotating animations, 3D
transforms, auto-scrolling carousels, auto-playing video, bouncing springs.

**Generally safe:** opacity/crossfade transitions, color changes, simple
state changes without spatial movement.

**Flash/seizure thresholds (WCAG 2.3.1, Level A):** no more than 3 flashes
per second; red flashing is especially dangerous. Prefer smooth transitions
over binary on/off toggling of visibility.

## Switch Control

If the app is accessible to VoiceOver, it's mostly accessible to Switch
Control already. What helps further: standard `Button` controls (scanned
automatically), a clear `accessibilityLabel` (shown in the Switch Control
menu), `accessibilityCustomActions` (appear as menu items), and adequate
touch targets (44×44pt minimum).

## Voice Control

Voice Control overlays element names from `accessibilityLabel`; users say
"Tap [label]" to activate. Use `accessibilityInputLabels` for short,
speakable alternatives:

```swift
Button { } label: { Image(systemName: "gearshape") }
    .accessibilityLabel("Settings with 3 notifications")
    .accessibilityInputLabels(["Settings", "Preferences", "Gear"])
// first label shown in the Voice Control overlay — keep it short and speakable
```

Rules: keep labels speakable (no abbreviations, no developer jargon); make
labels unique on screen (Voice Control can't disambiguate duplicates); the
first input label supersedes `accessibilityLabel` for display;
`accessibilityInputLabels` also improves Full Keyboard Access's "Find"
feature (Cmd+F). WCAG 2.5.3 (Label in Name) requires the `accessibilityLabel`
to *contain* the visible text — a button labeled "Submit" can't have an
accessible name of "Send form".

## Full Keyboard Access

Tab navigates between focusable elements, Space/Enter activates. All
standard controls are keyboard-focusable by default; `.focusable()` (iOS
17+) makes non-inputs focusable. Known limitation: `@FocusState` does not
move Full Keyboard Access focus to non-`TextField` elements. Keep a 3:1
contrast focus indicator (WCAG 2.4.13) and a logical tab order matching
visual order.

`@FocusState` (keyboard input focus) and `@AccessibilityFocusState`
(VoiceOver/AT focus) are separate systems — moving one does not move the
other, and `@AccessibilityFocusState` needs an async delay to work
programmatically:

```swift
@FocusState private var keyboardFocus: Field?
@AccessibilityFocusState private var voFocus: Field?

func showError() {
    keyboardFocus = .email
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
        voFocus = .error  // VoiceOver focus needs an async delay
    }
}
```

## Android and Flutter

*(Supplementary — not from the EPAM source; general platform knowledge.)*

**Android**: respect `Settings.Global.ANIMATOR_DURATION_SCALE` /
`ValueAnimator.areAnimatorsEnabled()` the same way iOS respects
`isReduceMotionEnabled` — when the user has disabled animator duration,
replace transitions with an immediate state change or crossfade rather than
skipping the transition silently. Android's alternative-input surface is
**Switch Access** (parallel to iOS Switch Control) and **Voice Access**
(parallel to Voice Control) — both rely on the same `contentDescription`/
`Semantics` labeling this skill's control references already require, so no
separate labeling work is needed beyond what §controls/*.md covers.

**Flutter**: check `MediaQuery.of(context).disableAnimations` (reflects both
iOS Reduce Motion and Android's animator-duration-scale setting through one
flag) before running implicit/explicit animations, and provide a
non-animated fallback. Flutter has no separate Switch Control/Switch Access
API of its own — on each platform it rides the native accessibility tree, so
a well-labeled `Semantics` tree is sufficient for both.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku; Android/Flutter section is supplementary, not sourced from EPAM.
