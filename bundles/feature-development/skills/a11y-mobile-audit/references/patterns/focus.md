# Focus

### #a11y - Native Accessibility Acceptance Criteria

How to test focus

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Focus visibly moves to all controls
   - Visible focus indicator: Is clearly shown around controls in a logical order

2. Test mobile screenreader gestures

   - Swipe: Initial focus moves to a logical place on screen, then to all meaningful images, text and controls
   - Doubletap: Activates controls

3. Listen to screenreader output on all devices

   - Initial focus: Starts in a logical place (ex: back button, close button, top of screen)
   - Navigate: Focus moves left to right, top to bottom in a logical order
   - Visible focus indicator: Is clearly shown around content when tapped or with navigation gestures

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/focus](/native-criteria/patterns/focus)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test focus

GIVEN THAT I am on a screen with focus

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB" key, "ARROW" keys, or "CTRL+TAB" keys   
      - THEN the focus should visibly move to all controls   
         - AND a visible focus indicator should be clearly shown around controls in a logical order   

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate   
      - THEN the initial focus should move to a logical place on the screen   
         - AND focus should then move to all meaningful images, text, and controls   
   - WHEN I double-tap a control   
      - THEN the control should be activated   

3. Scenario: Test screen reader output on all devices

   - WHEN the screen reader starts   
      - THEN the initial focus should start in a logical place (e.g., back button, close button, or top of the screen)   
   - WHEN I navigate through the content   
      - THEN the focus should move left to right, top to bottom in a logical order   
         - AND a visible focus indicator should be clearly shown around content when tapped or navigated using gestures

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size up to 200% in the device settings   
      - THEN the focus should visibly move to all controls without losing information 
         - AND a visible focus indicator should be clearly shown around controls in a logical order without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/focus](/native-criteria/patterns/focus)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/patterns/focus.md
