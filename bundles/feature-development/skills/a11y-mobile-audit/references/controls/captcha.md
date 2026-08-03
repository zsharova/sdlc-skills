# Captcha

### #a11y - Native App Accessibility Acceptance Criteria

How to test a captcha

1. Test keyboard only, then screen reader + keyboard actions

    - Tab, arrow keys or ctl+tab: Focus visibly moves to the captcha button
    - Spacebar: Activates the captcha on iOS and Android
    - Enter: Activates the button on Android

2. Test mobile screenreader gestures

    - Swipe: Focus moves to the interactive elements, expresses its state, if applicable
    - Doubletap: Activates the button

3. Listen to screenreader output on all devices

    - Name: Purpose is clear (ex: "Captcha")
    - Role: Identifies itself as a button or image button, if interactive
    - Group: n/a
    - State: Expresses its state (disabled/dimmed)

4. Device OS Settings

    - Text resize: n/a

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/captcha](/native-criteria/controls/captcha)

## Gherkin
### #a11y - Native App Accessibility Acceptance Criteria

How to test a captcha

GIVEN THAT I am on a screen with a captcha

1. Scenario: Test keyboard actions 

    - WHEN the user presses Tab, Arrow Keys, or Ctrl+Tab 
        - THEN the focus must visibly move to the captcha button 
    - WHEN the user presses Spacebar and or Enter 
        - THEN the button is activated 

2. Scenario: Test mobile screen reader gestures 

    - WHEN the user swipes to interactive elements 
        - THEN focus must move sequentially to the captcha button 
             - AND the screen reader must announce the state of the captcha button (e.g., enabled or disabled) 
    - WHEN the user performs a double-tap gesture 
        - THEN the captcha button must activate 

3. Scenario: Test screen reader output on all devices 

    - WHEN a screen reader reads the button 
        - THEN its name should clearly describe its purpose, captcha 
            - AND its role should be identified as a button or image button in iOS and as a button or "double tap to activate" in Android 
            - AND its state (DISABLED/DIMMED) should be expressed if applicable 

4. Scenario: Test device OS settings for text resize 

    - WHEN a user adjusts text resizing settings up to 200% 
        - THEN text resizing does not apply to the captcha functionality (n/a) 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/captcha](/native-criteria/controls/captcha)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/captcha.md
