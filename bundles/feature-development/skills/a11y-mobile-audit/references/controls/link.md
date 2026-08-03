# Link

### #a11y - Native App Accessibility Acceptance Criteria

How to test a link

1. Test keyboard only, then screen reader + keyboard actions

   - Tab, arrow keys, or ctrl+tab: Focus visibly moves to the link
   - Spacebar: Activates on iOS and Android
   - Enter: Activates on Android

2. Test mobile screenreader gestures

   - Swipe: Focus moves to the element, expresses its name, role (state, if applicable)
   - Rotor/talkback menu: Links can be navigated to and activated from the Rotor/TalkBack menu or by focus/double tap. Only one way is required. Known issue: Links do not currently appear in iOS Rotor using SwiftUI. 
   - Doubletap: Activates the link

3. Listen to screenreader output on all devices

   - Name: Purpose and destination is clear
   - Role: Identifies itself as a link
   - Group: n/a
   - State: Expresses its state if applicable (disabled/dimmed)

4. Device OS settings

   - Text resize: Text can resize up to 200% without losing information

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/link](/native-criteria/controls/link)

## Gherkin
### #a11y - Native App Accessibility Acceptance Criteria

How to test a link

GIVEN THAT I am on a screen with a link

1. Scenario: Test keyboard actions 

   - WHEN I press the "TAB", "ARROW KEYS", or "CTRL+TAB" keys 
      - THEN the focus should visibly move to the link 
   - WHEN I press the "SPACEBAR" key 
      - THEN the link should be activated on iOS and Android 
   - WHEN I press the "ENTER" key 
      - THEN the link should be activated on Android  

2. Scenario: Test mobile screen reader gestures 

   - WHEN I swipe to navigate to the link 
      - THEN the focus should move to the link 
         - AND the link's name, role, and state (if applicable) should be expressed 
   - WHEN I use the Rotor/TalkBack menu 
      - THEN the link should be navigable and activatable from the Rotor/TalkBack menu or by focus and double-tap 
         - AND at least one method should work 
   - WHEN I double-tap the link 
      - THEN the link should be activated 
          - AND KNOWN ISSUE: Links do not currently appear in iOS Rotor using SwiftUI 

3. Scenario: Test screen reader output on all devices 

   - WHEN a screen reader reads the link 
      - THEN its name should clearly describe its purpose and destination 
         - AND its role should be identified as a link 
         - AND its state (DISABLED/DIMMED) should be expressed, if applicable  

4. Scenario: Test device OS settings for text resize 

   - WHEN I adjust the device text resize setting to 200% 
      - THEN the text of the link should resize up to 200% without losing information 

Full information: [https://www.magentaa11y.com/#/native-criteria/controls/link](/native-criteria/controls/link)

## Flutter testing notes
Flutter apps render through the platform's native accessibility tree on both iOS and Android, so the VoiceOver/TalkBack scenarios above apply unchanged — there is no separate "Flutter screen reader" to test.

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/native/controls/link.md
