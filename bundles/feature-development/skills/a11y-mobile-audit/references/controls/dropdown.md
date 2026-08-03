# Dropdown

### #a11y - Native Accessibility Acceptance Criteria

How to test a dropdown

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the Dropdown
   - Spacebar: Selects and opens the Dropdown on iOS and Android
   - Enter: Selects and opens the Dropdown on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role, value & state (expanded or collapsed)
   - Doubletap: Selects and opens Dropdown

3. Listen to screenreader output on all devices

   - Name: Purpose is clear and matches any visible label
   - Role: Identifies itself as a button in iOS and "double tap to activate" in Android
   - Group: Visible label is grouped or associated with the Dropdown in a single swipe
   - State: Expresses its state (disabled/dimmed). Expanded/collapsed states are announced on the elements that close or open the dropdown

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/dropdown](/native-criteria/controls/dropdown)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a dropdown

GIVEN THAT I am on a screen with a dropdown

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" keys 
      - THEN the focus should visibly move to the dropdown 
   - WHEN I press the "ESCAPE" key 
      - THEN the dropdown should close and return focus to the button that launched it 
   - WHEN I press the "SPACEBAR" key 
      - THEN the dropdown should be selected and opened on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the dropdown should be selected and opened on Android 

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate to the dropdown 
      - THEN the focus should move to the dropdown 
         - AND the dropdown's name, role, value, and state (EXPANDED or COLLAPSED) should be expressed 
   - WHEN I double-tap the dropdown 
      - THEN the dropdown should be selected and opened

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the dropdown 
      - THEN its name should clearly describe its purpose and match any visible label 
         - AND its role should be identified as a button in iOS and as "double tap to activate" in Android 
         - AND its visible label should be grouped or associated with the dropdown in a single swipe 
         - AND its state (DISABLED/DIMMED) should be expressed 
         - AND the EXPANDED or COLLAPSED state should be announced on the elements that close or open the dropdown

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200%
      - THEN the text on the dropdown should resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/dropdown](/native-criteria/controls/dropdown)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/dropdown.md
