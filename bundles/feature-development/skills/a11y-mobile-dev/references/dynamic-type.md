# Dynamic Type / Text Scaling — iOS, Android, Flutter

Supports `a11y-mobile-dev`. Over 25% of iOS users change their preferred text
size — supporting it is required for WCAG 1.4.4 (Resize Text) on every
platform. The iOS section below is the deep reference; Android and Flutter
get a shorter parallel section since their scaling models are simpler.

## iOS — Dynamic Type

Always use text styles instead of hardcoded sizes:

```swift
// Bad — never scales
Text("Welcome").font(.system(size: 24))

// Good — scales with user preference
Text("Welcome").font(.title)
```

Style mapping:

| Text style | Default size | Weight | Typical use |
|---|---|---|---|
| `.largeTitle` | 34pt | Regular | Screen titles |
| `.title` | 28pt | Regular | Major section headings |
| `.title2` | 22pt | Regular | Sub-section headings |
| `.title3` | 20pt | Regular | Minor headings |
| `.headline` | 17pt | Semibold | Emphasized body text |
| `.body` | 17pt | Regular | Main content |
| `.callout` | 16pt | Regular | Secondary content |
| `.subheadline` | 15pt | Regular | Captions, metadata |
| `.footnote` | 13pt | Regular | Footnotes, fine print |
| `.caption` | 12pt | Regular | Labels, timestamps |
| `.caption2` | 11pt | Regular | Smallest readable text |

**Custom fonts in SwiftUI** auto-scale when given `relativeTo:`:
```swift
Text("Hello").font(.custom("Georgia", size: 24, relativeTo: .headline))  // scales
Text("Fixed").font(.custom("Georgia", fixedSize: 24))                    // does not
```

**UIFontMetrics for custom fonts (UIKit)**:
```swift
let customFont = UIFont(name: "Avenir-Medium", size: 17)!
label.font = UIFontMetrics(forTextStyle: .body).scaledFont(for: customFont)
label.adjustsFontForContentSizeCategory = true  // critical: enables live updating
label.numberOfLines = 0                          // allow text wrapping
```
`adjustsFontForContentSizeCategory` only works on `UILabel`, `UITextField`,
and `UITextView` — not arbitrary custom views.

**`@ScaledMetric` for non-text dimensions** (icons, spacing, avatars):
```swift
@ScaledMetric(relativeTo: .body) private var imageSize: CGFloat = 40
Image("logo").resizable()
    .aspectRatio(contentMode: .fit)
    .frame(width: imageSize, height: imageSize)
```
SF Symbols scale automatically with `.font()` — don't `.resizable()` them.
Decorative images should not scale; consider hiding them at accessibility
sizes instead.

**Layout adaptation at accessibility sizes** — horizontal layouts break when
text gets very large; switch to vertical:
```swift
// iOS 16+, automatic
ViewThatFits {
    HStack { content }
    VStack { content }
}

// iOS 16+, preserves state
@Environment(\.dynamicTypeSize) var dynamicTypeSize
let layout = dynamicTypeSize.isAccessibilitySize
    ? AnyLayout(VStackLayout()) : AnyLayout(HStackLayout())
layout { content }
```
UIKit equivalent: check
`traitCollection.preferredContentSizeCategory.isAccessibilityCategory` in
`traitCollectionDidChange` and flip `stackView.axis`.

**ScrollView for large text** — always ensure scrollability at accessibility
sizes, or use `ViewThatFits(in: .vertical)` to scroll only when needed.

**Large Content Viewer** — for controls that can't scale (toolbars, tab
bars), cap normal scaling and show a magnified version on long press:
```swift
LocationButton()
    .dynamicTypeSize(...DynamicTypeSize.xxxLarge)
    .accessibilityShowsLargeContentViewer {
        Label("Recenter", systemImage: "location")
    }
```

**Size categories** (12, smallest to largest): `extraSmall`, `small`,
`medium`, `large` (system default), `extraLarge`, `extraExtraLarge`,
`extraExtraExtraLarge`, then five `accessibility*` categories where
`isAccessibilityCategory` (UIKit) / `isAccessibilitySize` (SwiftUI) becomes
true. Restrict range only when truly necessary (e.g. `dynamicTypeSize(...DynamicTypeSize.xxxLarge)` on a compact toolbar item backed by a Large Content Viewer) — Apple explicitly discourages unduly limiting text size.

## Android and Flutter — text scaling

*(Supplementary — not from the EPAM source; general platform knowledge.)*

**Android**: define text sizes in `sp` (scale-independent pixels), never
`dp`/`px`, so they scale with the user's font-size setting
(`Settings > Display > Font size`) and, separately, with
`Configuration.fontScale`. In Compose, `TextUnit.sp` is the default for
`fontSize` — avoid hardcoding `TextUnit.Unspecified` overrides that bypass
scaling. Layouts should use `wrap_content`/weight-based sizing rather than
fixed heights so scaled text doesn't clip, matching the iOS "avoid fixed
frames" rule above.

**Flutter**: `MediaQuery.textScaler` (replaces the deprecated
`textScaleFactor`) reflects the platform's font-scale setting; `Text` widgets
respect it by default. Use `MediaQuery.withNoTextScaling` deliberately (icons,
fixed-size badges) rather than accidentally opting whole subtrees out of
scaling. As with iOS's `ViewThatFits`, prefer flexible layouts
(`Wrap`, `Flexible`, `SingleChildScrollView`) over fixed-height containers so
scaled text doesn't clip.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku; Android/Flutter section is supplementary, not sourced from EPAM.
