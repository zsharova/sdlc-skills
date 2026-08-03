# Button

### #a11y - Native App Accessibility Acceptance Criteria

How to test a button

1. Test keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to the button
   - Spacebar: Activates on iOS and Android
   - Enter: Activates on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role (state, if applicable)
   - Doubletap: Activates the button


3. Listen to screenreader output on all devices

   - Name: Purpose is clear and matches visible label
   - Role: Identifies as a button in iOS and button or "double tap to activate" in Android
   - Group: Visible label is grouped or associated with the button in a single swipe
   - State: Expresses its state (disabled/dimmed)

4. Test device settings

    - Text resize: Text can resize up to 200% without losing information

Full information: https://www.magentaa11y.com/#/native-criteria/controls/button

## Gherkin
### #a11y - Native App Accessibility Acceptance Criteria

How to test a button

GIVEN THAT I am on a screen with a button

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" 
      - THEN the focus should visibly move to the button 
   - WHEN I press the "SPACEBAR" key 
      - THEN the button should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the button should be activated on Android 

2. Scenario: Test mobile screen reader gestures 

   - WHEN I swipe to navigate to the button 
      - THEN the focus should move to the button 
          - AND the button's name, role, and state (if applicable) should be expressed 
   - WHEN I double-tap the button 
      - THEN the button should be activated 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the button 
      - THEN its name should clearly describe its purpose and match the visible label 
        - AND its role should be identified as a button in iOS and as a button or "double tap to activate" in Android 
        - AND its visible label should be grouped or associated with the button in a single swipe 
        - AND its state (DISABLED/DIMMED) should be expressed if applicable 

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200% 
      - THEN the text on the button should resize up to 200% without losing information 

Full information: https://www.magentaa11y.com/#/native-criteria/controls/button

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/button.md
