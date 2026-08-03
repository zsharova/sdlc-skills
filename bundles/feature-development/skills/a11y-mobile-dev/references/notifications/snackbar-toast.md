# Snackbar / Toast

How to test a snackbar/toast

## iOS
   - There is no native snackbar or toast element for iOS. The notes below are suggestions and accessibility guidance.

### Developer notes

   - Snackbars and Toasts provide lightweight feedback about an operation
   - They show a brief, non-critical message at the bottom of the screen on mobile
   - Snackbar or Toast must be dynamically announced by the screen reader without moving focus to it
   - Display only one Snackbar or Toast at a time
   - A Snackbar or toast can be timed to disappear, since the message is not critical information
   - If the Snackbar persists, it should be in the swipe order for the screen reader user and tab order for the keyboard user
   - All text in the Snackbar and Toast must be announced by the screen reader
   - A Snackbar may contain an interactive element. Please follow the Native Button guidance for the accessibility needs of a Call To Action (CTA).
   - Toasts do not contain an interactive element

### Dynamic Announcement example

   - "Copied to clipboard" (Non-critical information in timed-to-disappear snackbar)

## Android
### General Notes

   - Snackbars and Toasts provide lightweight feedback about an operation
   - They show a brief non-critical message at the bottom of the screen on mobile
   - Snackbar or Toast must be dynamically announced by the screen reader without moving focus to it
   - Display only one Snackbar or Toast at a time
   - A Snackbar or Ttoast can be timed to disappear, since the message is not critical information
   - If the Snackbar persists, it should be in the swipe order for the screen reader user and tab order for the keyboard user
   - All text in the Snackbar and Toast must be announced by the screen reader
   - A Snackbar may contain an interactive element. Please follow the Native Button guidance for the accessibility needs of a Call To Action (CTA).
   - Toasts do not contain an interactive element

### Focus

   - For Jetpack Compose Snackbar, when there is an action available, the default accessibility behavior allows screen reader to dynamically read the message and the action
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
      - `Modifier.focusOrder()` needs to be used in combination with `FocusRequester` to define focus order
      - `Modifier.onFocusEvent()`, `Modifier.onFocusChanged()` can be used to observe the changes to focus state
      - `FocusRequester` allows to request focus to individual elements with in a group of merged descendant views
      - Example: To customize the focus events
         - step 1: define the focus requester prior. `val (first, second) = FocusRequester.createRefs()`
         - step 2: update the modifier to set the order. `modifier = Modifier.focusOrder(first) { this.down = second }`
         - focus order accepts following values: up, down, left, right, previous, next, start, end
         - step 3: use `second.requestFocus()` to gain focus

### Code Example

   - **Jetpack Compose**

```java
val snackbarHostState = remember { SnackbarHostState() }
val scope = rememberCoroutineScope()
Scaffold(
    snackbarHost = { SnackbarHost(snackbarHostState) },
    content = { innerPadding ->
        Button(
          modifier = Modifier.padding(innerPadding),
          onClick = {
            // show snackbar as a suspend function
            scope.launch {
                snackbarHostState.showSnackbar("Snackbar Message")
            }
          }) {
              Text(text = "Snackbar")
          }
    }
)
```

### Dynamic Announcement example

   - "Copied to clipboard" (Non-critical information in timed-to-disappear snackbar)

## Flutter
### Developer notes
- Flutter's `SnackBar` (shown via `ScaffoldMessenger.of(context).showSnackBar()`) uses `Semantics(liveRegion: true)` internally so its content is announced without moving focus away from the current screen — this matches the non-modal, non-focus-stealing behavior snackbars/toasts require on both iOS and Android.
- Do not wrap a `SnackBar`'s content in `scopesRoute`/`namesRoute` the way a Dialog would — a snackbar is a transient live-region announcement, not a focus-trapping modal.
- For a custom toast (e.g. an `Overlay` entry rather than `SnackBar`), replicate this explicitly with `Semantics(liveRegion: true, label: '...')` or a direct `SemanticsService.announce()` call.
- If the snackbar includes an action button (e.g. "Undo"), that button needs its own reachable `Semantics(button: true)` — a screen reader user should be able to navigate to it before it auto-dismisses, so avoid overly short display durations.

### Dynamic Announcement example
- "Item deleted, Undo, button" — announced via live region without disturbing current focus

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/notifications/snackbar-toast.md
