# iOS — AI Failure Patterns

Supports `a11y-mobile-dev` (this skill) and its audit sibling `a11y-mobile-audit`,
which uses the same severity labels (Critical/High/Medium/Low) in its findings.

AI coding assistants systematically produce inaccessible iOS code — this is the
reference for the 11 documented failure modes, each with a vulnerable/correct
code pair and how to detect it in a diff.

## F1: `onTapGesture` instead of `Button`

**Severity:** Critical
**Impact:** Element completely invisible to VoiceOver, Switch Control, Full Keyboard Access, and visionOS eye tracking.
**Detection:** Search for `.onTapGesture` on views that perform actions.

```swift
// Bad — invisible to all assistive technology
Image(systemName: "heart.fill")
    .onTapGesture { toggleFavorite() }

// Good — Button gets .isButton trait automatically
Button(action: toggleFavorite) {
    Image(systemName: "heart.fill")
}
.accessibilityLabel("Add to favorites")
```

`onTapGesture` adds a gesture recognizer but does **not** add the `.isButton`
accessibility trait. VoiceOver skips the element entirely during linear
navigation; Switch Control can't scan to it; Voice Control can't target it.
This is the single most common and most damaging AI failure — see
[../controls/button.md](../controls/button.md) for the full button pattern.

## F2: Hardcoded font sizes

**Severity:** High
**Impact:** Text doesn't scale for 25%+ of users who change their text size setting.
**Detection:** Search for `.font(.system(size:`.

```swift
// Bad — fixed size
Text("Welcome").font(.system(size: 24))

// Good — Dynamic Type text style
Text("Welcome").font(.title)
```

```swift
// Bad — UIKit fixed size
label.font = UIFont.systemFont(ofSize: 16)

// Good — UIKit Dynamic Type with live updating
label.font = UIFont.preferredFont(forTextStyle: .body)
label.adjustsFontForContentSizeCategory = true
label.numberOfLines = 0
```

Text-style mapping:

| Old pattern | Dynamic Type replacement |
|---|---|
| `.system(size: 34)` | `.largeTitle` |
| `.system(size: 28)` | `.title` |
| `.system(size: 22)` | `.title2` |
| `.system(size: 20)` | `.title3` |
| `.system(size: 17)` | `.headline` (bold) or `.body` |
| `.system(size: 16)` | `.callout` |
| `.system(size: 15)` | `.subheadline` |
| `.system(size: 13)` | `.footnote` |
| `.system(size: 12)` | `.caption` |
| `.system(size: 11)` | `.caption2` |

See [../dynamic-type.md](../dynamic-type.md) for the full pattern, including
`@ScaledMetric` for non-text dimensions and layout adaptation.

## F3: Hardcoded colors

**Severity:** High
**Impact:** Text invisible or unreadable in dark mode; may also fail contrast requirements.
**Detection:** Search for `.foregroundColor(.black)`, `.foregroundColor(.white)`, `.background(Color.white)`, `.background(Color.black)`, hex literals for text/backgrounds.

```swift
// Bad — invisible in dark mode
Text("Hello").foregroundColor(.black)
    .background(Color.white)

// Good — semantic colors that adapt
Text("Hello").foregroundStyle(.primary)
    .background(Color(.systemBackground))
```

Semantic color mapping:

| Hardcoded | Semantic replacement |
|---|---|
| `.black` (text) | `.primary` |
| `.gray` (text) | `.secondary` |
| `.white` (background) | `Color(.systemBackground)` |
| `.init(white: 0.95)` (bg) | `Color(.secondarySystemBackground)` |
| `.init(white: 0.9)` (bg) | `Color(.tertiarySystemBackground)` |

See [../color-visual.md](../color-visual.md) for contrast ratios and dark-mode testing.

## F4: Deprecated API

**Severity:** High
**Impact:** Deprecated patterns that lost accessibility improvements present in their modern replacements.
**Detection:** Search for `.foregroundColor(`, `.cornerRadius(`, `NavigationView`.

```swift
// Bad — deprecated
.foregroundColor(.blue)
.cornerRadius(10)
NavigationView { }          // deprecated since iOS 16

// Good — modern equivalents
.foregroundStyle(.blue)
.clipShape(.rect(cornerRadius: 10))
NavigationStack { }
```

## F5: No accessibility labels on image buttons

**Severity:** Critical
**Impact:** VoiceOver reads the raw SF Symbol name or just "Button" with no context.
**Detection:** `Button` containing only `Image(systemName:)` without `.accessibilityLabel`.

```swift
// Bad — VoiceOver reads "plus" or "Button"
Button(action: addItem) { Image(systemName: "plus") }

// Good — meaningful label
Button(action: addItem) { Image(systemName: "plus") }
    .accessibilityLabel("Add item")
```

Label quality rules:
- Describe the action, not the icon — "Add item" not "Plus icon".
- Don't include "button" — "Add item" not "Add item button".
- Update the label when state changes — "Add to favorites" / "Remove from favorites".
- Provide context for identical icons — "Add peanut butter" not just "Add".

## F6: `GeometryReader` abuse and fixed frames

**Severity:** High
**Impact:** Layout breaks at accessibility text sizes, different screen sizes, and zoom.
**Detection:** Search for `GeometryReader` and `.frame(width:` with hardcoded pixel values.

```swift
// Bad — fixed frame prevents Dynamic Type growth
Text("Price: $9.99")
    .frame(width: 120, height: 44)

// Good — let the text grow, keep only a minimum for touch target
Text("Price: $9.99")
    .frame(minWidth: 44, minHeight: 44)
```

Avoid `GeometryReader` unless a custom layout calculation genuinely needs it.
Avoid fixed `frame(width:height:)`; use flexible layout with `minWidth`/
`minHeight` for touch targets only.

## F7: No accessibility on custom controls

**Severity:** Critical
**Impact:** Custom controls appear as static text or are invisible to VoiceOver.
**Detection:** Custom views with tap handling but no accessibility modifiers.

```swift
// Bad — AI-generated custom toggle, no accessibility
HStack {
    Text("Dark Mode")
    Circle()
        .fill(isDarkMode ? .blue : .gray)
        .onTapGesture { isDarkMode.toggle() }
}

// Good — accessible custom toggle
Button(action: { isDarkMode.toggle() }) {
    HStack {
        Text("Dark Mode")
        Circle().fill(isDarkMode ? .blue : .gray)
    }
}
.accessibilityElement(children: .combine)
.accessibilityAddTraits(.isToggle)
.accessibilityValue(isDarkMode ? "On" : "Off")
```

Required accessibility for any custom control: `accessibilityLabel` (what is
this), `accessibilityValue` (current state — "On"/"Off", "50%", "3 of 5"),
`accessibilityTraits` (what kind — `.button`, `.adjustable`, `.isToggle`),
`accessibilityAdjustableAction` (sliders/steppers — increment/decrement on
swipe up/down), `accessibilityCustomActions` (multi-action elements).

## F8: `accessibilityIdentifier` vs. `accessibilityLabel` confusion

**Severity:** Critical
**Impact:** Element has a test identifier but VoiceOver reads nothing useful.

```swift
// Bad — using identifier where a label is needed
button.accessibilityIdentifier = "favorite_button"
// VoiceOver: says nothing useful — the identifier is not read aloud

// Bad — developer jargon as label
button.accessibilityLabel = "btn_favorite_123"
// VoiceOver: "btn underscore favorite underscore one two three"

// Good — both set independently
button.accessibilityLabel = "Add to favorites"     // for VoiceOver
button.accessibilityIdentifier = "favoriteButton"  // for UI tests
```

## F9: Missing system-preference checks

**Severity:** High
**Impact:** App ignores the user's accessibility settings — animation, transparency, and color-only indicators all persist regardless of system settings.
**Detection:** `withAnimation` without a `reduceMotion` check; `.blur`/`.ultraThinMaterial` without a `reduceTransparency` check.

```swift
// Bad — ignores reduce motion
withAnimation(.spring()) { showDetails.toggle() }

// Good — respects reduce motion
@Environment(\.accessibilityReduceMotion) var reduceMotion
withAnimation(reduceMotion ? .none : .spring()) { showDetails.toggle() }
```

Environment values to check, and when:

| Setting | Check when… |
|---|---|
| `accessibilityReduceMotion` | Any `withAnimation`, `.transition`, `.spring()`, parallax, auto-scroll |
| `accessibilityReduceTransparency` | `.blur`, `.ultraThinMaterial`, any visual effect |
| `legibilityWeight` (Bold Text) | Custom font weights that might need bolder variants |
| `colorSchemeContrast` (Increase Contrast) | Custom colors that might need higher-contrast variants |
| `accessibilityDifferentiateWithoutColor` | Color-only status indicators (red/green/yellow) |
| `dynamicTypeSize` | Layout that needs HStack→VStack at accessibility sizes |
| `accessibilityInvertColors` | Photos/videos/maps that should not be inverted |

See [../motion-input.md](../motion-input.md) for the full pattern.

## F10: Assigning traits instead of inserting (UIKit)

**Severity:** Critical
**Impact:** Destroys existing traits — a button becomes non-button, breaking VoiceOver interaction.
**Detection:** Search for `.accessibilityTraits =` (assignment, not `.insert`/`.union`).

```swift
// Bad — DESTROYS the button trait, no longer announced as button
myButton.accessibilityTraits = .selected

// Good — insert/union preserves existing traits
myButton.accessibilityTraits.insert(.selected)

// Good — SwiftUI Add/RemoveTraits
Button("Option A") { }
    .accessibilityAddTraits(.isSelected)  // keeps .isButton
```

Common combinations: `tab.accessibilityTraits = [.button, .selected]` is fine
at initial setup only — subsequent state changes must use
`tab.accessibilityTraits.insert(.selected)`. A disabled link uses
`link.accessibilityTraits.insert(.notEnabled)` to preserve `.link`.

## F11: Hiding disabled controls from VoiceOver

**Severity:** Critical
**Impact:** The user doesn't know the control exists and can't discover what's needed to enable it.

```swift
// Bad — user has no idea this button exists
submitButton.isAccessibilityElement = false

// Good — UIKit notEnabled trait makes VoiceOver say "dimmed"
submitButton.accessibilityTraits.insert(.notEnabled)
// VoiceOver: "Submit, dimmed"

// Good — SwiftUI .disabled() automatically adds the notEnabled trait
Button("Submit") { submitForm() }
    .disabled(!formIsValid)
// VoiceOver: "Submit. Dimmed. Button."
```

When a VoiceOver user hears "Submit, dimmed" they know: the button exists,
they can't activate it yet, and they should look for what to fill in to
enable it. Hiding the button removes all three signals.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku.