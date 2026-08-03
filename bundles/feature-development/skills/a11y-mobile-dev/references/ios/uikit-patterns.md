# iOS — UIKit Accessibility Patterns

UIKit patterns for custom elements, containers, trait combinations, and
modal handling — the manual work SwiftUI's standard controls give you for
free.

## Custom-drawn views (`UIAccessibilityElement`)

For views that draw their own content (`drawRect`, `CALayer`, Metal) and
have no natural child views for VoiceOver to walk:

```swift
class GraphView: UIView {
    override var isAccessibilityElement: Bool {
        get { false }  // container, not an element itself
        set { }
    }

    override var accessibilityElements: [Any]? {
        get {
            dataPoints.enumerated().map { index, point in
                let element = UIAccessibilityElement(accessibilityContainer: self)
                element.accessibilityLabel = "\(point.label)"
                element.accessibilityValue = "\(point.value) units"
                element.accessibilityFrame = UIAccessibility.convertToScreenCoordinates(
                    point.frame, in: self
                )
                element.accessibilityTraits = .staticText
                return element
            }
        }
        set { }
    }
}
```

Use `UIAccessibility.convertToScreenCoordinates(_:in:)` for the frame — this
is the canonical API for custom-drawn views because the system expects
screen-space coordinates when hit-testing the VoiceOver cursor.
(`accessibilityFrameInContainerSpace` is an alternative, but it requires the
view to sit in a standard container hierarchy, which custom-drawn graphs
often don't.)

## `UIAccessibilityContainer`

```swift
class CustomContainer: UIView {
    func accessibilityElementCount() -> Int { items.count }
    func accessibilityElement(at index: Int) -> Any? { items[index] }
    func index(ofAccessibilityElement element: Any) -> Int {
        items.firstIndex { $0 === element as AnyObject } ?? NSNotFound
    }
    override var accessibilityContainerType: UIAccessibilityContainerType {
        get { .list }  // .none, .dataTable, .list, .landmark, .semanticGroup
        set { }
    }
}
```

## Trait combinations

```swift
tab.accessibilityTraits = [.button, .selected]           // initial setup only
link.accessibilityTraits = [.link, .notEnabled]           // initial setup only
headerButton.accessibilityTraits = [.header, .button]     // initial setup only
customSlider.accessibilityTraits = [.adjustable]          // must implement increment/decrement
timerLabel.accessibilityTraits = [.updatesFrequently]     // live-updating value
```

**Critical rule:** assignment (`=`) is only safe the first time you declare
a control's complete trait set. Any subsequent state change must use
`.insert()`/`.remove()` — assignment on a state change destroys every other
trait already present:

```swift
// GOOD — state change
tab.accessibilityTraits.insert(.selected)
tab.accessibilityTraits.remove(.selected)

// BAD — state change: this LOSES .button
tab.accessibilityTraits = .selected
```

## Table/collection views

```swift
headerLabel.accessibilityTraits = .header

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
    cell.accessibilityLabel = item.title
    cell.accessibilityHint = "Double tap to view details"
    cell.accessibilityCustomActions = [
        UIAccessibilityCustomAction(name: "Delete", target: self, selector: #selector(deleteItem(_:))),
        UIAccessibilityCustomAction(name: "Share", target: self, selector: #selector(shareItem(_:)))
    ]
    return cell
}

cell.contentView.shouldGroupAccessibilityChildren = true
```

Don't set `isAccessibilityElement = false` on cells, and don't put multiple
visible buttons inside a cell — use `accessibilityCustomActions` instead of
stacking tappable subviews VoiceOver has to swipe through individually.

## Custom properties

| Property | Purpose |
|---|---|
| `accessibilityActivationPoint` | Set when the accessible hit area doesn't match the visible tap target |
| `accessibilityPath` | A `UIBezierPath` hit area for non-rectangular controls |
| `accessibilityLanguage` | Per-element language override — VoiceOver switches voice/pronunciation for that element |
| `accessibilityNavigationStyle` | `.automatic` (default) / `.combined` / `.separate` |

```swift
customControl.accessibilityActivationPoint = CGPoint(x: 50, y: 50)
circularButton.accessibilityPath = UIBezierPath(ovalIn: circularButton.bounds)
spanishQuote.accessibilityLanguage = "es"
```

## Attributed labels (mixed-language content)

```swift
let attr = NSMutableAttributedString(string: "The word is hola")
attr.addAttribute(.accessibilitySpeechLanguage, value: "es", range: NSRange(location: 13, length: 4))
label.accessibilityAttributedLabel = attr
```

## Navigation and grouping

```swift
// Groups children into one swipe stop instead of letting VoiceOver
// interleave elements from adjacent containers
containerView.shouldGroupAccessibilityChildren = true

// Completely overrides default reading order — you now own ALL elements
// in this container, including any you forgot to list
view.accessibilityElements = [headerLabel, contentView, actionButton, footerLabel]

// Custom action with an icon (iOS 14+)
override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
    get { [UIAccessibilityCustomAction(name: "Pin", image: UIImage(systemName: "pin"),
                                        target: self, selector: #selector(pinAction))] }
    set { }
}
```

## Rules

- Insert/remove traits on any state change; only assign the full trait set
  once, at initial setup.
- Custom-drawn views: `isAccessibilityElement = false` on the container,
  one `UIAccessibilityElement` per logical sub-region, screen coordinates
  via `UIAccessibility.convertToScreenCoordinates(_:in:)`.
- Never hide table/collection cells (`isAccessibilityElement = false`);
  never stack multiple visible buttons in a cell — use custom actions.
- Overriding `accessibilityElements` makes you responsible for every
  element in that container — a missed one silently disappears from
  VoiceOver.

---
Adapted from the EPAM `epam-ios-accessibility` skill (v1.0.5) by Ruslan Popesku.
