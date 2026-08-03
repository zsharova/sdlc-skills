# Animation

### #a11y - Native Accessibility Acceptance Criteria

How to test an animation

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys or ctl+tab: N/A

2. Test mobile screenreader gestures

   - Swipe: Focus moves to and from the animation if in the swipe order

3. Listen to screenreader output on all devices

   - Focus: If meaningful, animation is focused and its meaning announced via alt text

4. Test device settings

   - Text resize: N/A

Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/animation](/native-criteria/patterns/animation)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test an animation

GIVEN THAT I am on a screen with an animation

1. Scenario: Test keyboard actions

   - WHEN I am using keyboard navigation
      - THEN keyboard interaction is N/A

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate
      - THEN focus should move to and from the animation, if it is in the swipe order 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader is active 
      - THEN if the animation is meaningful, it should receive focus
         - AND its meaning should be announced via alt text 

4. Scenario: Test device OS settings for text resize

   - WHEN I have increased text size in device settings
      - THEN text resize interaction is N/A 
 
Full information: [https://www.magentaa11y.com/#/native-criteria/patterns/animation](/native-criteria/patterns/animation)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/patterns/animation.md
