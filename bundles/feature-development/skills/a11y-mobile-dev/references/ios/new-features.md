# iOS — New Accessibility APIs by Release

New APIs and platform features across recent iOS releases. Skim this when
targeting a specific iOS version or reviewing code that could adopt a newer
API than the one it uses.

## iOS 17 (WWDC23)

**`AccessibilityNotification` (SwiftUI-native)** replaces the UIKit
`UIAccessibility.post(notification:argument:)` pattern:

```swift
AccessibilityNotification.Announcement("Loading complete").post()

// With priority, to avoid being interrupted by other speech
var announcement = AttributedString("Camera Active")
announcement.accessibilitySpeechAnnouncementPriority = .high
AccessibilityNotification.Announcement(announcement).post()

AccessibilityNotification.ScreenChanged(newFocusElement).post()
AccessibilityNotification.LayoutChanged(element).post()
```

Other iOS 17 additions: `.accessibilityAddTraits(.isToggle)` for custom
toggle-like controls; `.contentShape(.accessibility, Shape)` for a custom
VoiceOver hit area; `.accessibilityDirectTouch(options:)` to bypass
VoiceOver navigation for an element; `.accessibilityZoomAction { }` for
custom zoom handling; `.focusable()` to make a non-input view
keyboard-focusable for Full Keyboard Access.

User-facing features: Assistive Access (simplified interface mode),
Personal Voice (150 sentences to build an AI voice clone), Live Speech
(type-to-speak in person and on calls), Point and Speak (LiDAR + ML reads
labels on physical objects).

## iOS 18 (WWDC24)

```swift
// Conditional labels — no if/else branch needed
Image(systemName: isFavorite ? "star.fill" : "star")
    .accessibilityLabel("Super Favorite", isEnabled: isFavorite)

// View-based label composition
.accessibilityLabel { Text(rating) + Text(existingLabel) }

// AppIntent triggered directly from a VoiceOver custom action
.accessibilityAction(named: "Favorite") { FavoriteBeachIntent(beach: beach) }
```

User-facing features: Eye Tracking (front camera only, no extra hardware),
Music Haptics, Vocal Shortcuts (custom voice commands for any action),
Vehicle Motion Cues (reduces motion sickness), enhanced Braille display
support.

## iOS 26 (WWDC 2025)

Platform features: Accessibility Nutrition Labels (App Store metadata
declaring accessibility features), Accessibility Reader (system-wide
reading mode for any app), Brain-Computer Interface protocol for Switch
Control, Personal Voice needing only 10 phrases (down from 150), Head
Tracking with custom facial-expression action mapping, an Assistive Access
API for building tailored simplified experiences, Voice Control
programming mode for Xcode, and Liquid Glass prompting system-wide Reduce
Transparency improvements.

## Quick modifier-to-version lookup

| Modifier | iOS | Purpose |
|---|---|---|
| `.accessibilityAddTraits(.isToggle)` | 17+ | Toggle trait for custom toggle controls |
| `.contentShape(.accessibility, Shape)` | 17+ | Custom hit path for VoiceOver |
| `.accessibilityDirectTouch()` | 17+ | Bypass VoiceOver navigation for element |
| `.accessibilityZoomAction` | 17+ | Custom zoom behavior |
| `AccessibilityNotification` | 17+ | Native SwiftUI notification posting |
| `.focusable()` | 17+ | Make non-inputs keyboard-focusable |
| `.accessibilityLabel(_, isEnabled:)` | 18+ | Conditional labels |
| `.accessibilityLabel { View }` | 18+ | View-based label composition |
| `.accessibilityAction(named:, intent:)` | 18+ | AppIntent-backed actions |

## Rules

- Prefer `AccessibilityNotification.Announcement(...).post()` over the
  legacy `UIAccessibility.post(notification: .announcement, argument:)` on
  iOS 15+ — it's type-safe and avoids a background-thread crash class the
  legacy API is prone to. Targeting iOS 14 or below still requires the
  legacy API, dispatched on the main thread.
- Before hand-rolling a workaround for a toggle, zoom gesture, or
  conditional label, check this file first — iOS 17/18 likely already
  shipped a purpose-built modifier.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku.