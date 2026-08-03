# Checkbox

### #a11y - Native Accessibility Acceptance Criteria

How to test a checkbox

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the checkbox
   - Spacebar: Activates on iOS and Android
   - Enter: Activates on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role, state
   - Doubletap: Checkbox toggles between checked and unchecked states

3. Listen to screenreader output on all devices

   - Name: Name describes the purpose of the control and matches the visible label
   - Role: Identifies itself as a checkbox in Android and a Button or checkbox in iOS
   - Group: Visible label can be grouped with the checkbox in a single swipe
   - State: Expresses its state (disabled/dimmed, checked, not checked, selected, unselected)

4. Test device settings

   - Text resize: Text label can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/checkbox](/native-criteria/controls/checkbox)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a checkbox

GIVEN THAT I am on a screen with a checkbox

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" keys 
      - THEN the focus should visibly move to the checkbox 
   - WHEN I press the "SPACEBAR" key 
      - THEN the checkbox should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the checkbox should be activated on Android

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate to the checkbox 
      - THEN the focus should move to the checkbox 
         - AND the checkbox's name, role, and state should be expressed 
   - WHEN I double-tap the checkbox 
      - THEN the checkbox should toggle between CHECKED and NOT CHECKED states 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the checkbox 
      - THEN its name should describe the purpose of the control and match the visible label 
         - AND its role should be identified as a checkbox in Android and as a button or checkbox in iOS 
         - AND its visible label should be grouped with the checkbox in a single swipe 
         - AND its state (DISABLED/DIMMED, CHECKED, NOT CHECKED, SELECTED, UNSELECTED) should be expressed

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200% 
      - THEN the text label should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/checkbox](/native-criteria/controls/checkbox)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/checkbox.md
