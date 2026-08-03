# Decorative Images

### #a11y - Native Accessibility Acceptance Criteria

How to test a decorative image or icon

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: Image does not receive focus, N/A

2. Test mobile screenreader gestures

   - Swipe: The screenreader ignores the image completely, N/A

3. Listen to screenreader output on all devices

   - Role: The image is not annoucned and is ignored completely, N/A

4. Test device settings

   - Text resize: N/A

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/image-decorative](/native-criteria/patterns/image-decorative)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a decorative image or icon

GIVEN THAT I am on a screen with a decorative image or icon

1. Scenario: Test keyboard actions

   - WHEN I use keyboard navigation
      - THEN the image or icon is ignored, N/A

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate 
      - THEN focus moves to the image or icon
         - I HEAR that the image or icon is ignored, N/A

3. Scenario: Test screen reader output on all devices

   - WHEN I use a screenreader (Talkback, VoiceOver) 
      - AND I swipe to browse to an image or icon
         - I HEAR the image or icon is ignored, N/A

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size in device settings 
      - AND I swipe to browse to an image or icon
         - I HEAR the image or icon is ignored, N/A

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/image-decorative](/native-criteria/patterns/image-decorative)

## Flutter testing notes
- Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.
- Use the Flutter DevTools Semantics Debugger to confirm decorative `Image`/`Icon` widgets were actually excluded (`ExcludeSemantics` or `excludeFromSemantics: true`) — a missed decorative image is otherwise easy to overlook since it may not have distinguishable text for a tester to notice during a swipe pass.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/patterns/image-decorative.md
