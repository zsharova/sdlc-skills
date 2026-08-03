# Loading Icon

### #a11y - Native Accessibility Acceptance Criteria

How to test a loading icon

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: N/A

2. Test mobile screenreader gestures

   - Swipe: Focus can move to the loading icon, but it should not be necessary because “loading” is announced dynamically

3. Listen to screenreader output on all devices

   - Name: “Loading” is announced dynamically

4. Test device settings

   - Text resize: N/A

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/loading-icon](/native-criteria/patterns/loading-icon)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a loading icon

GIVEN THAT I am on a screen with a loading icon

1. Scenario: Test keyboard actions

   - WHEN I am using keyboard navigation
      - THEN keyboard interaction is N/A 

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate
      - THEN focus moves to the loading icon
         - BUT it should not be necessary because "Loading" is announced dynamically

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader is active
      - THEN "Loading" should be announced dynamically

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size in device settings
      - THEN text resize interaction is N/A 

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/loading-icon](/native-criteria/patterns/loading-icon)

## Flutter testing notes
- Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.
- Confirm via the Semantics Debugger that the spinner announces once on appearance/completion rather than repeatedly on every animation frame — a common Flutter-specific regression when `liveRegion` is applied directly to an animating widget instead of a stable wrapper.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/patterns/loading-icon.md
