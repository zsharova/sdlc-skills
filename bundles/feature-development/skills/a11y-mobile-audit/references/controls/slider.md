# Slider

### #a11y - Native Accessibility Acceptance Criteria

How to test a slider

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, ctr+tab or arrow keys: Focus moves visibly to the input
   - Right and left arrow-keys: Increase / decrease value one step

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the input
   - Ios and android-swipe-up/down: Increase/decrease slider value one step
   - Android-volume or swipe up/down: Increase/decrease slider value one step


3. Listen to screenreader output on all devices

   - Name: Name describes the purpose of the control and matches the visible label
   - Role: Identifies itself as "adjustable" in iOS and "Slider" in Android
   - Group: Group label with control, when possible to give the slider a programmatic name
   - State: Expresses its current value, if applicable

4. Test device settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/slider](/native-criteria/controls/slider)

## Gherkin
### #a11y - Native Accessibility Acceptance Criteria

How to test a slider

GIVEN THAT I am on a screen with a slider

1. Scenario: Test keyboard actions

   - WHEN I press the "TAB", "CTRL+TAB", or "ARROW KEYS" 
      - THEN the focus should visibly move to the slider input 
   - WHEN I press the "RIGHT ARROW" key 
      - THEN the slider value should increase by one step 
   - WHEN I press the "LEFT ARROW" key 
      - THEN the slider value should decrease by one step 

2. Scenario: Test mobile screen reader gestures

   - WHEN I swipe to navigate to the slider input 
      - THEN the focus should move to the slider input 
   - WHEN I swipe up or down on iOS or Android 
      - THEN the slider value should increase or decrease by one step 
   - WHEN I use the volume controls or swipe up/down on Android 
      - THEN the slider value should increase or decrease by one step 

3. Scenario: Test screen reader output on all devices

   - WHEN a screen reader reads the slider 

      - THEN its name should clearly describe its purpose and match the visible label 
         - AND its role should be identified as "adjustable" in iOS and as "Slider" in Android 
         - AND its label should be programmatically grouped with the control, if possible 
         - AND its current value should be expressed, if applicable 

4. Scenario: Test device OS settings for text resize

   - WHEN I adjust the device text resize setting to 200%
      - THEN the text label should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/slider](/native-criteria/controls/slider)

## Flutter testing notes
- Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.
- Verify the Flutter `Slider`/`RangeSlider` has an explicit `label` set (via `Semantics` or a wrapping label) before testing — an unlabeled Flutter slider is the most common regression, and it fails silently in a way that's easy to miss without a screen reader on.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/slider.md
