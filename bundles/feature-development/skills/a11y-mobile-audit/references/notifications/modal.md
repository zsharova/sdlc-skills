# Modal

### #a11y - Native Accessibility Acceptance Criteria

How to test a modal

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to any interactive element
   - Escape: The modal closes and returns focus to the button that launched it
   - Space: Any buttons or links are activated on iOS and Android
   - Enter: Any buttons or links are activated on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves into the modal, confined within the modal
   - Doubletap: This typically activates most elements (alternative custom actions may be implemented)
   - Group: n/a

3. Listen to screenreader output on all devices

   - Name: The modal itself is not interactive. Any close button label should describe the close action
   - Role: Any CTA in the modal announces as a button
   - State: n/a

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/segmented-control](/native-criteria/controls/segmented-control)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a modal

GIVEN THAT I am on a screen with a modal

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB" key, "ARROW" keys, or "CTRL+TAB" keys
        - THEN the focus should visibly move to any interactive element within the modal 
   - WHEN I press the "ESCAPE" key
        - THEN the modal should close
            - AND focus should return to the button that launched it
   - WHEN I press the "SPACEBAR" key
        - THEN any buttons or links should be activated on iOS and Android
   - WHEN I press the "ENTER" key
        - THEN any buttons or links should be activated on Android 

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate 
        - THEN focus should move into the modal and remain confined within it 
   - WHEN I double-tap an element 
        - THEN it should be activated unless an alternative custom action is implemented 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader is active 
        - THEN the modal itself should not be announced as interactive 
            - AND any close button label should clearly describe the close action 
            - AND any call-to-action (CTA) elements within the modal should be announced as a button 

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size in device settings
        - THEN text should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/segmented-control](/native-criteria/controls/segmented-control)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/notifications/modal.md
