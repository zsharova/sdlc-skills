# Carousel

### #a11y - Native Accessibility Acceptance Criteria

How to test a carousel

1. Test keyboard only, then screen reader + keyboard actions

      - Tab, arrow keys or ctl+tab: Focus visibly moves to the next item

      - Spacebar: Activates button or interactive slide on iOS and Android

      - Enter: Activates button or interactive slide on Android

2. Test mobile screenreader gestures

      - Swipe right: Focus moves to the next element 

      - 3 finger swipe: Focus moves to the next slide (iOS)

      - 2 finger swipe: Focus moves to the next slide (Android)

      - 1 finger swipe up or down or other custom actions: Focus 
      moves to the next slide on iOS

      - Doubletap: Activates the button

3. Listen to screenreader output on all devices

      - Name: Purpose of each item is clear and matches visible text.  Index should be announced in Android and iOS

      - Role: Identifies as a button in iOS and "double tap to activate" in Android

      - Role: Identifies as "adjustable" with custom actions (iOS)

      - Group: n/a

      - State: Expresses a button's state (disabled/dimmed)

4. Test device settings

      - Text resize: Text can resize up to 200% without losing information


Full information: [https://www.magentaa11y.com/#/native-criteria/controls/carousel](/native-criteria/controls/carousel)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a carousel

GIVEN THAT I am on a screen with a carousel

1. Scenario: Test keyboard actions

    - WHEN the user presses "TAB", Arrow Keys, or "CTRL+TAB" 
        - THEN the focus must visibly move to the next carousel item 
    - WHEN the user presses "SPACEBAR" on iOS or Android 
        - THEN the button or interactive slide must activate 
    - WHEN the user presses "ENTER" on Android 
        - THEN the button or interactive slide must activate 

2. Scenario: Test mobile screen reader gestures

    - WHEN the user swipes right 
        - THEN focus must move to the next interactive element 
    - WHEN the user performs a 3-finger swipe on iOS 
        - THEN focus must move to the next slide 
    - WHEN the user performs a 2-finger swipe on Android 
        - THEN focus must move to the next slide 
    - WHEN the user performs a 1-finger swipe up or down or a custom action on iOS 
        - THEN focus must move to the next slide 
    - WHEN the user performs a double-tap 
        - THEN the button must activate 

3. Scenario: Test screen reader output on all devices

    - WHEN the user swipes through the elements 
        - THEN each item must be announced with the following attributes: 
          - AND the Name must clearly describe the purpose and match the visible text 
          - AND the Role must be identified as "Button" in iOS and "Double tap to activate" in Android 
          - AND the Role must be identified as "Adjustable" with custom actions in iOS 
          - AND the Group must be marked as not applicable (N/A) 
          - AND the State must announce the button’s state (e.g., DISABLED/DIMMED) 

4. Scenario: Test device OS settings for text resize

    - WHEN a user adjusts text resizing settings up to 200% 
        - THEN all text must remain readable without loss of information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/carousel](/native-criteria/controls/carousel)

## Flutter testing notes
- Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.
- Before manual AT testing, inspect the carousel's semantics tree with the Flutter DevTools Semantics Debugger to confirm off-screen `PageView` items are excluded and the container announces "Carousel" and is adjustable, since a `PageView`-based carousel has no built-in semantics to rely on.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/carousel.md
