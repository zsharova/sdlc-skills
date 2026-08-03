# WCAG 2.2 AA → Native Platform API Mapping

Supports `a11y-mobile-dev` and its audit sibling `a11y-mobile-audit` (this is
the widest-scope reference in the pair — the audit skill's stack-detection
checklist cites this file directly rather than duplicating it). Maps WCAG 2.2
AA success criteria to iOS, Android, and Flutter APIs, and identifies what's
automatically handled by standard controls vs. what needs manual work, per
platform.

## iOS

### Automatically handled by standard UIKit/SwiftUI controls

| WCAG | Criterion | Why it's handled |
|---|---|---|
| 4.1.2 | Name, Role, Value | Standard controls auto-set traits, labels, values |
| 2.5.2 | Pointer Cancellation | UIKit fires on `.touchUpInside`, not down |
| 2.1.1/2.1.2 | Keyboard | Standard controls work with VoiceOver, Switch Control, keyboards |
| 3.1.1 | Language | iOS respects `NSLocale` |
| 3.2.1/3.2.2 | On Focus/Input | Standard controls don't trigger context changes |
| 1.3.2 | Meaningful Sequence | Default left-to-right, top-to-bottom reading order |

### Requires manual implementation

| WCAG | Criterion | iOS API | Severity | Notes |
|---|---|---|---|---|
| 1.1.1 | Non-text Content | `accessibilityLabel` on images; `accessibilityHidden(true)` for decorative | Critical | The #1 AI failure — always missing |
| 1.3.1 | Info & Relationships | `.header` trait; `accessibilityElements`; `.accessibilityElement(children:)` | High | Headings, grouping, tables |
| 1.3.4 | Orientation | Don't lock orientation in Info.plist | Medium | |
| 1.3.5 | Input Purpose | `.textContentType(.emailAddress)` on TextFields | Medium | Enables autofill |
| 1.4.1 | Use of Color | `accessibilityDifferentiateWithoutColor` + icons/text | High | Never color-only indicators |
| 1.4.3 | Contrast (Minimum) | 4.5:1 normal, 3:1 large; `accessibilityContrast` trait | High | Test both light and dark mode |
| 1.4.4 | Resize Text | `preferredFont(forTextStyle:)` + `adjustsFontForContentSizeCategory` | High | AI always uses hardcoded sizes |
| 1.4.10 | Reflow | Dynamic Type accessibility sizes + HStack→VStack adaptation | High | `ViewThatFits`, `AnyLayout` |
| 1.4.11 | Non-text Contrast | 3:1 for UI components and borders | Medium | Design concern |
| 1.4.13 | Content on Hover | Popovers/tooltips must be dismissible and persistent | Medium | |
| 2.2.1 | Timing | Timeout warnings with extend/disable options | Medium | |
| 2.2.2 | Pause, Stop, Hide | `isReduceMotionEnabled`; pause controls for auto-playing | High | |
| 2.3.1 | Three Flashes | No content >3 flashes/sec | Critical | Seizure risk |
| 2.4.3 | Focus Order | `accessibilityElements`; `.accessibilitySortPriority` | High | |
| 2.4.6 | Headings and Labels | `.accessibilityAddTraits(.isHeader)` | High | Enables rotor navigation |
| 2.5.1 | Pointer Gestures | Single-tap alternatives for multi-touch gestures | Medium | |
| 2.5.3 | Label in Name | `accessibilityLabel` must contain visible text | High | Critical for Voice Control |
| 2.5.7 | Dragging (new in 2.2) | Single-tap alternatives; `accessibilityCustomActions` | Medium | |
| 2.5.8 | Target Size (new in 2.2) | Min 24×24pt (Apple HIG: 44×44pt) | Medium | |
| 3.3.1 | Error Identification | `AccessibilityNotification.Announcement` for errors | High | |
| 3.3.8 | Auth (new in 2.2) | `.password` textContentType; enable paste; biometric auth | Medium | Support password managers |

### WCAG 2.2 new criteria affecting iOS

| Criterion | Requirement | iOS implementation |
|---|---|---|
| 2.4.11 Focus Not Obscured | Sticky headers/footers must not cover the focused element | Scroll-to-visible on focus |
| 2.4.13 Focus Appearance | 3:1 contrast focus indicator for custom controls | Custom focus rings on non-standard controls |
| 2.5.7 Dragging Movements | Non-drag alternatives for all drag operations | `accessibilityCustomActions` for reorder |
| 2.5.8 Target Size Minimum | 24×24pt minimum (Apple HIG: 44×44pt) | `.frame(minWidth: 44, minHeight: 44)` |
| 3.2.6 Consistent Help | Help in the same position across screens | Consistent help-button placement |
| 3.3.7 Redundant Entry | Auto-populate previously entered info | `textContentType` for autofill |
| 3.3.8 Accessible Auth | Support password managers, paste, biometrics | `.password` textContentType, no paste blocking |

### Does not apply to native iOS

| WCAG | Criterion | Why not applicable |
|---|---|---|
| 2.4.1 | Bypass Blocks | Within a single native app screen, not applicable — no rotor equivalent needed |
| 4.1.1 | Parsing | "Always satisfied" per the 2023 WCAG errata; iOS isn't HTML |
| 1.4.12 | Text Spacing | Only for markup-based content (WebViews), not native views |

## Android

### Automatically handled by standard Views/Compose controls

| WCAG | Criterion | Why it's handled |
|---|---|---|
| 4.1.2 | Name, Role, Value | `Button`/Compose `Button` set role/state semantics by default |
| 2.5.2 | Pointer Cancellation | Standard controls fire on `ACTION_UP` inside bounds, not `ACTION_DOWN` |
| 2.1.1/2.1.2 | Keyboard | Standard Views/Compose controls work with TalkBack, Switch Access, external keyboards |
| 3.1.1 | Language | Android respects the device locale |
| 1.3.2 | Meaningful Sequence | Default top-to-bottom, traversal-order reading |

### Requires manual implementation

| WCAG | Criterion | Android API | Severity | Notes |
|---|---|---|---|---|
| 1.1.1 | Non-text Content | `contentDescription` (Views) / `Modifier.semantics { contentDescription }` (Compose) | Critical | Same #1 failure as iOS `accessibilityLabel` |
| 1.3.1 | Info & Relationships | `android:screenReaderFocusable` / `Modifier.semantics(mergeDescendants = true)` | High | Grouping, headings via `heading()` semantics property |
| 1.4.3 | Contrast (Minimum) | 4.5:1 normal, 3:1 large | High | Test both day and night theme |
| 1.4.4 | Resize Text | `sp` units, not `dp`/`px`; respect `Configuration.fontScale` | High | See `dynamic-type.md`'s Android section |
| 2.4.6 | Headings and Labels | `Modifier.semantics { heading() }` / accessibility heading traversal | High | No level concept, same limitation as iOS `.isHeader` |
| 2.5.7 | Dragging (new in 2.2) | Custom accessibility actions for non-drag reorder | Medium | `AccessibilityNodeInfo.AccessibilityAction` |
| 2.5.8 | Target Size Minimum | 48×48dp recommended (Material minimum), 24×24dp WCAG floor | Medium | |
| 3.3.8 | Auth (new in 2.2) | `autofillHints`, biometric prompt (`BiometricPrompt`) | Medium | |

### Does not apply to native Android

| WCAG | Criterion | Why not applicable |
|---|---|---|
| 2.4.1 | Bypass Blocks | Within a single native screen — TalkBack's reading-controls menu covers navigation instead |
| 4.1.1 | Parsing | Not HTML — no markup to validate |
| 1.4.12 | Text Spacing | Only relevant to WebViews, not native Views/Compose |

## Flutter

### Automatically handled by standard widgets

| WCAG | Criterion | Why it's handled |
|---|---|---|
| 4.1.2 | Name, Role, Value | Material/Cupertino widgets set `Semantics` (button/label/value) by default |
| 2.1.1/2.1.2 | Keyboard | Standard widgets integrate with the platform's native accessibility tree (VoiceOver/TalkBack) and `FocusNode` traversal |
| 1.3.2 | Meaningful Sequence | Default widget-tree traversal order |

### Requires manual implementation

| WCAG | Criterion | Flutter API | Severity | Notes |
|---|---|---|---|---|
| 1.1.1 | Non-text Content | `Semantics(label:)` on icon-only controls; `ExcludeSemantics`/ decorative image handling | Critical | Same pattern as iOS/Android |
| 1.3.1 | Info & Relationships | `MergeSemantics`, `Semantics(header: true)` | High | No heading-level concept, same as iOS |
| 1.4.3 | Contrast (Minimum) | 4.5:1 normal, 3:1 large | High | Test both `Brightness.light` and `.dark` |
| 1.4.4 | Resize Text | Respect `MediaQuery.textScaler`, avoid fixed-size text containers | High | See `dynamic-type.md`'s Flutter section |
| 2.5.7 | Dragging (new in 2.2) | `Semantics(customSemanticsActions:)` for non-drag reorder | Medium | |
| 2.5.8 | Target Size Minimum | 48×48dp (Material) / 44×44pt (Cupertino) recommended, 24×24 WCAG floor | Medium | |

### Does not apply to native Flutter apps

| WCAG | Criterion | Why not applicable |
|---|---|---|
| 2.4.1 | Bypass Blocks | Same rationale as iOS/Android — no in-app "skip to content" concept |
| 4.1.1 | Parsing | Not HTML — no markup to validate |
| 1.4.12 | Text Spacing | Only relevant to an embedded `WebView` widget, not the Flutter widget tree |

## Decision table: system settings (all platforms)

| Setting | iOS check | Android/Flutter equivalent | Action when enabled |
|---|---|---|---|
| Text scaling | `dynamicTypeSize.isAccessibilitySize` | `Configuration.fontScale` / `MediaQuery.textScaler` | Switch HStack→VStack, add scroll |
| Increase Contrast | `contrast == .increased` | No direct Android equivalent; Flutter exposes iOS's `MediaQuery.highContrast` | Use 7:1 ratios, darker borders |
| Reduce Transparency | `reduceTransparency` | N/A on Android (no system blur-reduction toggle); Flutter has no direct equivalent | Solid backgrounds, no blur |
| Reduce Motion | `reduceMotion` | `Settings.Global.ANIMATOR_DURATION_SCALE` / `MediaQuery.disableAnimations` | Crossfade/disable animations |
| Differentiate w/o Color | `diffWithoutColor` | No system flag on Android/Flutter — apply unconditionally | Add shapes/icons to color indicators |
| Screen reader active | `voiceOverEnabled` | `AccessibilityManager.isTouchExplorationEnabled` / platform channel | Avoid disruptive UI changes (rare use) |

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku; Android and Flutter tables are supplementary, reasoned in parallel to the iOS mapping and not sourced from EPAM.
