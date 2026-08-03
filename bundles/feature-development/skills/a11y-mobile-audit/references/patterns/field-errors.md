# Field Errors

### #a11y - Native Accessibility Acceptance Criteria

How to test a field error

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Error message are not usually focusable and will not receive focus when using the keyboard

2. Test mobile screenreader gestures

   - Swipe: Error message are not usually focusable and will not receive focus when navigating via swipe

3. Listen to screenreader output on all devices

   - Error message: “Error” is usually announced as alt text for an error icon and the error message announcement follows it. Error message will be announced when user enters the text field with the error.

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/field-errors](/native-criteria/patterns/field-errors)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a field error

GIVEN THAT I am on a screen with a field error

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB" key, "ARROW" keys, or "CTRL+TAB" keys   
      - THEN the error message should not receive focus   

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate   
      - THEN the error message should not receive focus   

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the error   
      - THEN "ERROR" should be announced as alt text for an error icon   
         - AND the error message should be announced immediately after   
   - WHEN I navigate into the text field with the error   
      - THEN the error message should be announced   

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size up to 200% in the device settings   
      - THEN the text should resize without losing information   

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/field-errors](/native-criteria/patterns/field-errors)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/patterns/field-errors.md
