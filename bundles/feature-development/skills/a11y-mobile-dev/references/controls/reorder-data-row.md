# Reorder Data Row

How to test a reorder data row

## iOS
### General Notes

   - A common pattern on the reorder component is to offer a delete option. If this is the case, then the label is interactive, with the delete button appearing upon double tap. "Remove" is added prior to the label announcement programmatically.
   - The native component provides thorough instruction announcements that are very intuitive

### Name

   - Programmatic name describes the purpose of the control.
   - The programmatic name should match the visible text label
      - **Note:** Setting a programmatic name while a visible text label exists may cause VoiceOver to duplicate the announcement of the name. If this happens, hide the visible text label from VoiceOver recognition.

   - **UIKit**

      - You can programmatically set the visible label with `setTitle()`.
         - The _component title will overwrite the _component `accessibilityLabel`.
      - If a visible label is not applicable in this case, set the _component `accessibilityLabel` to the label of your choice.
         - To do this in Interface Builder, set the label using the Identity Inspector
      - To hide labels from VoiceOver programmatically, set the label's `isAccessibilityElement` property to `false`
      - To hide labels from VoiceOver using Interface Builder, uncheck `Accessibility Enabled` in the Identity Inspector.

   - **SwiftUI**

      - If no visible label, use view modifier `accessibilityLabel(_:)`.

### Role

   - When using non-native controls (custom controls), roles will need to be manually coded.
   - Follow code guidance for Button control. Ensure “draggable” is also announced.

### State 

   - **UIKit** 

      - For enabled: Set `isEnabled` to `true`.
      - For disabled: Set `isEnabled` to `false`. Announcement for disabled is "Dimmed".
         - If necessary, you may change the accessibility trait of the _component to `notEnabled`, but this may overwrite the current accessibility role of the control.

   - **SwiftUI**

      - For selected, use `accessibilityAddTraits(.isSelected)`.
      - For disabled, use view modifier `disabled()`.

### Focus

   - Use the device's default focus functionality. 
   - Consider how focus should be managed between child elements and their parent views.

   - **UIKit**

      - If VoiceOver is not reaching a particular element, set the element's `isAccessibilityElement` to `true`
         - **Note:** You may need to adjust the programmatic name, role, state, and/or value after doing this, as this action may overwrite previously configured accessibility.
      - Use `accessibilityViewIsModal` to contain the screen reader focus inside the modal.
      - To move screen reader focus to newly revealed content, use `UIAccessibility.post(notification:argument:)` that takes in `.screenChanged` and the newly revealed content as the parameter arguments.
      - To NOT move focus, but dynamically announce new content: use `UIAccessibility.post(notification:argument:)` that takes in `.announcement` and the announcement text as the parameter arguments.
      - `UIAccessibilityContainer` protocol: Have a table of elements that defines the reading order of the elements. 
 
   - **SwiftUI**

      - For general focus management that impacts both screen readers and non-screen readers, use the property wrapper `@FocusState` to assign an identity of a focus state.
         - Use the property wrapper `@FocusState` in conjunction with the view modifier `focused(_:)` to assign focus on a view with `@FocusState` as the source of truth.
         - Use the property wrapper `@FocusState`in conjunction with the view modifier `focused(_:equals:)` to assign focus on a view, when the view is equal to a specific value.
      - If necessary, use property wrapper `@AccessibilityFocusState` to assign identifiers to specific views to manually shift focus from one view to another as the user interacts with the screen with VoiceOver on.

### Announcement examples for editing Keyboards in Settings

   - "Remove, English US, button" (Option to delete row - Delete button appears and announces as "Delete, button"
   - "Reorder, English, button, draggable, double tap and hold, wait for the sound, then drag to rearrange" (on three-line reorder button)
   - "Moved below Spanish" (After double tap and hold and two-click sound is heard, slowly drag up or down, waiting for the row above or below to announce. Continue to slowly move up or down until moved to the desired location. Lift finger to complete task)

## Android
### General Notes

   - There is no specific native component for Reordering rows in Android. The experience is simply tap with three fingers to show custom actions (with screen reader on), double tap Actions and choose 1) Move down, 2) Move up, 3) Move to bottom, 4) Move to top or 5) Remove.
   - These are implemented as buttons. Please see the Button component for guidance.

### Focus

   - Only manage focus when needed. Primarily, let the device manage default focus
   - External keyboard tab order often follows the screen reader focus, but sometimes needs focus management


   - **Android Views**

      - `importantForAccessibility` makes the element visible to the Accessibility API
      - `android:focusable`
      - `android=clickable`
      - Implement an `onClick( )` event handler for keyboard, as well as `onTouch( )`
      - `nextFocusDown`
      - `nextFocusUp`
      - `nextFocusRight`
      - `nextFocusLeft`
      - `accessibilityTraversalBefore` (or after)
      - To move screen reader focus to newly revealed content: `Type_View_Focused`
      - To NOT move focus, but dynamically announce new content: `accessibilityLiveRegion`(set to polite or assertive)
      - To hide controls: `importantForAccessibility=false`
      - For a `ViewGroup`, set `screenReaderFocusable=true` and each inner object’s attribute to keyboard focus (`focusable=false`)

   - **Jetpack Compose**

      - `Modifier.focusTarget()` makes the component focusable
      - `Modifier.focusOrder()` needs to be used in combination with FocusRequesters to define focus order
      - `Modifier.onFocusEvent()`, `Modifier.onFocusChanged()` can be used to observe the changes to focus state
      - `FocusRequester` allows to request focus to individual elements with in a group of merged descendant views
      - Example: To customize the focus events
         - step 1: define the focus requester prior. `val (first, second) = FocusRequester.createRefs()`
         - step 2: update the modifier to set the order. `modifier = Modifier.focusOrder(first) { this.down = second }`
         - focus order accepts following values: up, down, left, right, previous, next, start, end
         - step 3: use `second.requestFocus()` to gain focus


### Custom Accessibility Action

   - When UI elements are customized and coded to look like a specific component say button, to ensure that name, role, state and action are all intact might need to update accessibility service and semantics.
   - Disclaimer: This customization would not be needed unless it is required to modify/add gestures or actions.
   - The Button class by default supplies all the necessary semantics to make it fully accessible.

   - **Android Views**

      - step 1: Create an accessibility service
      - step 2: Add the `FLAG_REQUEST_ACCESSIBILITY_BUTTON` flag in an AccessibilityServiceInfo object's `android:accessibilityFlags` attribute
      - step 3: To have a custom service register for the button's custom action callbacks, use `registerAccessibilityButtonCallback()`

   - **Jetpack Compose**

      - List of custom accessibility actions can be defined relatively easily in compose compared to Views using customActions. 
      - Example: `modifier = Modifier.semantics { customActions = listOf(CustomAccessibilityAction(label = "", action = { true }))}`

### Announcement examples

   - "Actions available. Please tap with three fingers to view" (Brings the TalkBack menu into view)
   - "TalkBack menu. Actions, in list, double tap to activate" (Usually first item in the TalkBack menu)
   - "Actions, Move down, in list, double tap to activate" (Performs chosen action)

## Flutter
Flutter's `ReorderableListView` provides default drag-handle semantics, but a screen-reader-only reorder path also needs an explicit alternative, matching the source's emphasis on non-drag reorder actions.

### Name
- Each row's `Semantics(label: ...)` should include its content, matching the Button guidance the source points to.

### Role
- Mark the row `Semantics(button: true)`, and additionally announce "draggable" via the `label`/`hint`, since Flutter has no separate "draggable" `SemanticsFlag` — matching the source's explicit call to also announce draggability.

### Custom Accessibility Action
- Add `CustomSemanticsAction`s ("Move up", "Move down") via `Semantics(customSemanticsActions: {...})` so TalkBack/VoiceOver users can reorder without a drag gesture — this is the direct Flutter equivalent of the Android custom-accessibility-action / iOS custom-rotor-action pattern for reorder rows.

### Focus
- After a reorder action fires, keep focus on the moved row (not reset to the top of the list) so the user can verify its new position and issue another move action if needed.

### Announcement examples for editing Keyboards in Settings
- "Label, button, draggable"
- "Moved to position 2 of 5" (via `SemanticsService.announce()` after a custom move action)

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/reorder-data-row.md
