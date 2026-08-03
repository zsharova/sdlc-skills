# Color & Visual Accessibility — iOS, Android, Flutter

Supports `a11y-mobile-dev`. Covers WCAG contrast ratios, color blindness,
dark mode, reduce transparency, and smart invert. The iOS section is the deep
reference (contrast math, environment values, asset-catalog variants);
Android and Flutter get a shorter parallel section.

## WCAG contrast ratios (all platforms)

| Level | Normal text | Large text (18pt+ or 14pt+ bold) | UI components |
|---|---|---|---|
| AA (required) | 4.5:1 | 3:1 | 3:1 |
| AAA (enhanced) | 7:1 | 4.5:1 | — |

Exemptions: placeholder text, disabled controls, decorative text, logos.
Large text is defined as 18pt (24px) regular or 14pt (18.67px) bold.

~8% of men and ~0.5% of women have some form of color vision deficiency
(red-green most common) — never use color as the sole indicator of state:

```swift
// Bad — status by color only, invisible to colorblind users
Circle().fill(isOnline ? .green : .red)

// Good — color + shape + text
HStack {
    Image(systemName: isOnline ? "checkmark.circle.fill" : "xmark.circle.fill")
    Text(isOnline ? "Online" : "Offline")
}
```

## iOS

**Programmatic contrast calculation:**
```swift
extension UIColor {
    var relativeLuminance: CGFloat {
        var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0
        getRed(&r, green: &g, blue: &b, alpha: nil)
        func linearize(_ c: CGFloat) -> CGFloat {
            c <= 0.03928 ? c / 12.92 : pow((c + 0.055) / 1.055, 2.4)
        }
        return 0.2126 * linearize(r) + 0.7152 * linearize(g) + 0.0722 * linearize(b)
    }

    func contrastRatio(with color: UIColor) -> CGFloat {
        let l1 = max(relativeLuminance, color.relativeLuminance)
        let l2 = min(relativeLuminance, color.relativeLuminance)
        return (l1 + 0.05) / (l2 + 0.05)
    }

    func meetsWCAG_AA(against bg: UIColor, isLargeText: Bool = false) -> Bool {
        contrastRatio(with: bg) >= (isLargeText ? 3.0 : 4.5)
    }
}
```

**Differentiate without color:**
```swift
@Environment(\.accessibilityDifferentiateWithoutColor) var diffWithoutColor
if diffWithoutColor {
    HStack {
        Image(systemName: "checkmark").foregroundStyle(.green)
        Text("Success")
    }
} else {
    Circle().fill(.green)
}
```

**Increased contrast** — `contrast == .increased` (SwiftUI) /
`traitCollection.accessibilityContrast == .high` (UIKit) means switching to
7:1 ratios and darker borders. Asset catalogs can define 4 variants (Any
Appearance, Dark Appearance, Any High Contrast, Dark High Contrast); system
colors auto-adapt.

**Reduce transparency:**
```swift
@Environment(\.accessibilityReduceTransparency) var reduceTransparency
.background(
    reduceTransparency
        ? AnyShapeStyle(Color(.systemBackground))
        : AnyShapeStyle(.ultraThinMaterial)
)
```
Replace `.ultraThinMaterial`, `.blur()`, and any translucent overlay with a
solid/opaque equivalent when this is enabled.

**Smart Invert Colors** — prevents inversion on real-world content (photos,
video, maps, user-generated content):
```swift
imageView.accessibilityIgnoresInvertColors = true        // UIKit
Image("photo").accessibilityIgnoresInvertColors(true)     // SwiftUI
```
Setting this on a parent cascades to all subviews. Use semantic system
colors (`.label`, `.systemBackground`) — they adapt correctly to inversion
without needing the override.

**Dark mode** — test contrast independently for both light and dark modes;
use semantic colors that auto-adapt; never hardcode `.black` for text or
`.white` for backgrounds.

| Purpose | UIKit | SwiftUI |
|---|---|---|
| Primary text | `.label` | `.primary` |
| Secondary text | `.secondaryLabel` | `.secondary` |
| Primary background | `.systemBackground` | `Color(.systemBackground)` |
| Secondary background | `.secondarySystemBackground` | `Color(.secondarySystemBackground)` |
| Grouped background | `.systemGroupedBackground` | `Color(.systemGroupedBackground)` |
| Separator | `.separator` | `Color(.separator)` |
| Tint/accent | `.tintColor` | `.accentColor` / `.tint` |

## Android and Flutter

*(Supplementary — not from the EPAM source; general platform knowledge.)*

**Android**: use theme attributes (`?attr/colorOnSurface`,
`?attr/colorSurface`) rather than hardcoded hex so day/night theme (`nightMode`
in `AppCompatDelegate`) switches colors automatically, mirroring iOS semantic
colors. Android has no direct "Increase Contrast" system toggle equivalent to
iOS — treat the AA 4.5:1/3:1 thresholds as the floor regardless. Differentiate
without color the same way: pair any red/green status indicator with an icon
or text label, since Android also has no built-in
"never use color alone" system setting to detect and react to.

**Flutter**: use `Theme.of(context).colorScheme` semantic roles
(`onSurface`, `surface`, `error`) instead of hardcoded `Color(0x...)` literals,
and drive dark mode via `ThemeMode.system` + `Brightness` so both Material and
Cupertino widgets adapt together. `MediaQuery.of(context).highContrast`
(iOS-only signal, exposed to Flutter) is the closest equivalent to iOS's
Increase Contrast — check it before hardcoding contrast ratios in custom
paint code.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku; Android/Flutter section is supplementary, not sourced from EPAM.
