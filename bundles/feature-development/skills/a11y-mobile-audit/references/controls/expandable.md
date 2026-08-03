# Expandable

### #a11y - Native Accessibility Acceptance Criteria

How to test a button that toggles an expandable region


1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the expandable button on Android
   - Tab or arrow keys: Focus visibly moves to the expandable button on iOS
   - Spacebar: Activates on iOS and Android
   - Enter: Activates on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role (state, if applicable)
   - Doubletap: Toggles the expandable button (and focus remains on the expandable button)

3. Listen to screenreader output on all devices

   - Name: Purpose is clear and matches visible label
   - Role: Identifies as an expandable button in iOS, and an expandable button or "double tap to activate" or "double tap to collapse” in Android
   - Group: Visible label is grouped or associated with the expandable button in a single swipe
   - State: Expresses its state (expanded/collapsed)

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/expandable](/native-criteria/controls/expandable)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test an expandable region

GIVEN THAT I am on a screen with an expandable region

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" keys 
      - THEN the focus should visibly move to the expandable region 
   - WHEN I press the "SPACEBAR" key 
      - THEN the expandable region should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the expandable region should be activated on Android 

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate to the expandable region 
      - THEN the focus should move to the expandable region 
         - AND the expandable region's name, role, and state (if applicable) should be expressed 
   - WHEN I double-tap the expandable region 
      - THEN the expandable region should be activated 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the expandable region 
      - THEN its name should clearly describe its purpose and match the visible label 
         - AND its role should be expressed 

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200%
      - THEN the text on the expandable region should resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/expandable](/native-criteria/controls/expandable)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/expandable.md
