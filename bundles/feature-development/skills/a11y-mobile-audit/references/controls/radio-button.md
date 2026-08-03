# Radio Button

### #a11y - Native App Accessibility Acceptance Criteria

How to test a radio button

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the radio button
   - Spacebar: Activates on iOS and Android
   - Enter: Activates on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role, and state
   - Doubletap: Toggles the radio button state

3. Listen to screenreader output on all devices

   - Name: Purpose is clear and matches any visible label
   - Role: Identifies itself as a button in iOS and radio button in Android
   - Group: Visible label can be grouped or associated with the radio button in a single swipe
   - State: Expresses its state (disabled/dimmed, iOS: checked/unchecked, selected/unselected. Android: checked/not checked)

4. Device OS Settings

   - Text resize: Text label can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/radio-button](/native-criteria/controls/radio-button)

## Gherkin
### #a11y - Native App Accessibility Acceptance Criteria

How to test a radio button

GIVEN THAT I am on a screen with a radio button

1. Scenario: Test keyboard actions 

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" 
      - THEN the focus should visibly move to the radio button 
   - WHEN I press the "SPACEBAR" key 
      - THEN the radio button should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the radio button should be activated on Android 

2. Scenario: Test mobile screen reader gestures 

   - WHEN I swipe to navigate to the radio button 
      - THEN the focus should move to the radio button 
         - AND the radio button's name, role, and state should be expressed 
   - WHEN I double-tap the radio button 
      - THEN the radio button state should toggle 

3. Scenario: Test screen reader output on all devices 

   - WHEN a screen reader reads the radio button 
      - THEN its name should clearly describe its purpose and match any visible label 
         - AND its role should be identified as a button in iOS and as a radio button in Android 
         - AND its visible label should be grouped or associated with the radio button in a single swipe 
         - AND its state (DISABLED/DIMMED, iOS: CHECKED/UNCHECKED, SELECTED/UNSELECTED, Android: CHECKED/NOT CHECKED) should be expressed 

4. Scenario: Test device OS settings for text resize 

   - WHEN I adjust the device text resize setting to 200% 
      - THEN the text label should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/radio-button](/native-criteria/controls/radio-button)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/radio-button.md
