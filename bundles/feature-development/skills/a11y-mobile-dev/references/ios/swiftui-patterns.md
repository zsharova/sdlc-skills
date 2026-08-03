# iOS — SwiftUI Accessibility Patterns

All SwiftUI accessibility modifiers and the component patterns built from
them. Read this when writing or reviewing SwiftUI screens.

## Core modifiers

```swift
// Label — for any element whose visible content doesn't say what it does
Button(action: share) { Image(systemName: "square.and.arrow.up") }
    .accessibilityLabel("Share")

// Hint — extra context read after a pause; starts with a verb
Button("Delete").accessibilityHint("Permanently removes this message")

// Value — current state of a control
Slider(value: $volume, in: 0...100)
    .accessibilityLabel("Volume")
    .accessibilityValue("\(Int(volume)) percent")

// Traits — add without removing what's already there
Text("Account Settings").font(.headline).accessibilityAddTraits(.isHeader)
Button("Option A") { }.accessibilityAddTraits(.isSelected)  // keeps .isButton

// Grouping
HStack { Text("Price"); Text("$9.99") }
    .accessibilityElement(children: .combine)          // "Price, $9.99"
HStack { Text("4"); Image(systemName: "star.fill") }
    .accessibilityElement(children: .ignore)
    .accessibilityLabel("4 out of 5 stars")

// Hide decorative content
Image("decorative-divider").accessibilityHidden(true)

// Reading order within a group (higher = read earlier)
VStack {
    ErrorBanner().accessibilitySortPriority(2)
    Title().accessibilitySortPriority(1)
    Content()                                          // default 0, read last
}

// Multi-action elements
.accessibilityCustomActions([
    .init(named: "Like") { likeItem(); return true },
    .init(named: "Share") { shareItem(); return true }
])

// Adjustable controls (sliders, steppers)
.accessibilityAdjustableAction { direction in
    switch direction {
    case .increment: value += 1
    case .decrement: value -= 1
    @unknown default: break
    }
}
```

## Less-common but load-bearing modifiers

- **`.accessibilityRepresentation { ... }`** — swaps in an alternative
  accessible view for a complex custom-drawn view (e.g. represent a custom
  gauge as a `Slider` for VoiceOver).
- **`.accessibilityChildren { ... }`** — supplies custom children for a
  container that draws its own content (`Canvas`), so each data point gets
  its own accessible element instead of the canvas reading as one blob.
- **`.accessibilityLabeledPair(role:id:in:)`** (iOS 17+) — links a separate
  label view to a nearby control across the view hierarchy, so VoiceOver
  announces them as one name+field pair instead of two independent elements.
- **`.accessibilityCustomContent(_:_:importance:)`** — extra info surfaced
  via the "More Content" rotor; `.high` importance is read automatically
  with the label, default importance requires rotor navigation.
- **`.accessibilityShowsLargeContentViewer { ... }`** — for controls that
  can't scale (toolbar/tab-bar items): pair with capping Dynamic Type at
  `.xxxLarge` so oversized text falls back to the long-press magnifier
  instead of breaking the layout.
- **`.accessibilityInputLabels([...])`** — short, speakable alternatives for
  Voice Control; the first label also improves Full Keyboard Access's
  Cmd+F "Find" feature.
- **`.accessibilityZoomAction` / `.accessibilityDirectTouch()`** (iOS 17+) —
  custom zoom handling and bypassing VoiceOver navigation for an element
  that needs direct-touch gestures.
- **`.contentShape(.accessibility, Shape)`** (iOS 17+) — a non-rectangular
  hit area for VoiceOver, independent of the visual hit-testing shape.
- **`.accessibilityLabel(_:isEnabled:)`** (iOS 18+) — a label that only
  applies conditionally, without an `if`/`else` branch.
- **`.accessibilityAction(named:intent:)`** (iOS 18+) — triggers an
  `AppIntent` directly from a VoiceOver custom action.

## Component patterns

- **`NavigationStack`** — standard `NavigationLink` auto-announces screen
  changes; set `.navigationTitle(...)` so VoiceOver has something to
  announce on appear.
- **`List`/`ForEach`** — `Section("Account") { ... }` headers get
  `.isHeader` automatically; a hand-written heading (`Text("Account")
  .font(.headline)`) needs `.accessibilityAddTraits(.isHeader)` added
  manually.
- **`.sheet` / `.alert` / `.confirmationDialog`** — all auto-handle modal
  focus containment and the VoiceOver escape gesture; nothing extra needed.
- **`TabView`** — auto-announces "Home, tab, 1 of 4"; `.badge(5)` auto-reads
  "5 items".
- **`Toggle`/`Slider`/`Picker`/`Stepper`** — standard controls
  auto-announce name/role/value; only add `.accessibilityLabel`/`.Value`
  when the visible content alone doesn't convey it (e.g. an unlabeled
  slider).
- **`Image`** — informative images get `.accessibilityLabel(...)`;
  decorative images use `Image(decorative:)` (preferred) or
  `.accessibilityHidden(true)`; SF Symbols are auto-labeled from their
  symbol name but that name is usually wrong — always override it.
- **`TextField`/`SecureField`** — a visible prompt is used as the label
  automatically; set `.textContentType(...)` for autofill regardless, and
  add an explicit `.accessibilityLabel` when the prompt text differs from
  what the label should say.
- **`Canvas`** — draws pixels with nothing for VoiceOver by default; pair
  `.accessibilityLabel` (overall description) with `.accessibilityChildren`
  (one element per data point).
- **Swift Charts** — has built-in accessibility (audio graphs, data
  summaries) out of the box; use `.accessibilityChartDescriptor(self)` only
  to customize the generated description.
- **Custom `Shape`/`Path`** — needs the full manual set: label, value, and
  `.contentShape(.accessibility, Shape)` for a correct hit area.

## Focus management

`@AccessibilityFocusState` is a **separate system** from `@FocusState`:
`@FocusState` controls keyboard input focus, `@AccessibilityFocusState`
controls what VoiceOver/AT reads next. Moving one does not move the other —
set both when an interaction needs both (e.g. showing a validation error).

```swift
enum FocusableField: Hashable { case email, password, error }
@AccessibilityFocusState private var accessFocus: FocusableField?

Text("Error: Invalid email")
    .accessibilityFocused($accessFocus, equals: .error)

// Programmatic focus change needs an async delay — VoiceOver focus
// doesn't reliably move within the same run-loop turn as the state change.
DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
    accessFocus = .error
}
```

## Rules

- Prefer a standard control (`Button`, `Toggle`, `Slider`, `Picker`) over a
  custom-built equivalent — it gets name/role/value/traits for free.
- `.accessibilityElement(children: .combine)` when child labels read
  naturally joined; `.ignore` + a custom label when they don't (`.combine`
  inserts pauses between joined labels, `.ignore` lets you write one
  coherent sentence).
- Never pair `.combine` with an interactive trait on the same container —
  `.isButton` plus `.combine` on the same element causes double-activation;
  use `.ignore` + custom label instead.
- Programmatic `@AccessibilityFocusState` changes need an async delay to
  actually move VoiceOver focus.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku.
