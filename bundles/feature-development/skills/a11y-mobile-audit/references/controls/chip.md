# Chip

### #a11y - Native Accessibility Acceptance Criteria

How to test a chip

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the chip
   - Space: Any elements inside are activated on iOS and Android
   - Enter: Any elements inside are activated on Android
2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its state, if applicable
   - Doubletap: Activates the chip

3. Listen to screenreader output on all devices

   - Name: Purpose is clear and matches the group visible label or chip label
   - Role: Identifies as a button in iOS and "double tap to activate" or checkbox in Android
   - Group: n/a
   - State: Expresses its state (selected/disabled/dimmed)

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/chip](/native-criteria/controls/chip)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a chip

GIVEN THAT I am on a screen with a chip

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" 
      - THEN the focus should visibly move to the chip 
   - WHEN I press the "SPACEBAR" key 
      - THEN any elements inside the chip should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN any elements inside the chip should be activated on Android

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate to the chip 
      - THEN the focus should move to the chip 
         - AND the chip's state should be expressed, if applicable 
   - WHEN I double-tap the chip 
      - THEN the chip should be activated

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the chip 
      - THEN its name should clearly describe its purpose and match the group visible label or chip label 
         - AND its role should be identified as a button in iOS and as "double tap to activate" or checkbox in Android 
         - AND its state (SELECTED, DISABLED, DIMMED) should be expressed if applicable 

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200% 
      - THEN the text label should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/chip](/native-criteria/controls/chip)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/chip.md
