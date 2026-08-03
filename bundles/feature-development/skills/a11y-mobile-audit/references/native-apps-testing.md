# Native Apps — Manual Tester Guide

Testing in native apps is essential to ensuring the app is accessible and
functional for all users. This is the manual-testing protocol `a11y-mobile-audit`
proposes for critical flows — see the skill's Testing & CI section.

## How to test with your screen reader

Use a screen reader, such as [TalkBack](https://support.google.com/accessibility/android/topic/3529932?hl=en&ref_topic=9078845&sjid=10047972329698138905-NC) (for Android) or [VoiceOver](https://support.apple.com/guide/iphone/turn-on-and-practice-voiceover-iph3e2e415f/ios) (for iOS).

- Swipe with one finger anywhere on the screen to the right to navigate through the screen. Swipe left navigates backwards.
- If custom actions are implemented, swipe up or down with one finger anywhere on the screen to perform the action.
- Scroll with two fingers for Android.
- Scroll with three fingers for iOS.
- Double tap to activate when the link is focused.
- Activate Rotor or the TalkBack menu to access a list of links to activate with double tap. (Only one way of accessing links is required: focus within the screen, or the context menus.)
- Note: currently a known bug in iOS — links cannot be accessed from the Rotor using SwiftUI.

### Screen reader shortcut one-time setup

**iOS VoiceOver**
1. Open Settings
2. Choose Accessibility
3. Choose Accessibility Shortcut
4. Select VoiceOver
5. To start or stop VoiceOver, press the power button three times (no need to go to Settings each time to turn the screen reader on and off)

**TalkBack**
1. Open Settings
2. Choose Accessibility
3. Choose Advanced Settings
4. Choose Volume up and down keys
5. Enable the toggle
6. To start or stop TalkBack, press both volume keys for 2-3 seconds (no need to go to Settings each time to turn the screen reader on and off)

### iOS Rotor and Android TalkBack Menu

**iOS Rotor setup**
1. With VoiceOver on, using two fingers on the screen (many prefer to make a pinching-gesture and rotate on the screen), rotate fingers to hear the options available in the rotor.
2. Keep rotating to cycle through the options, such as "headings".
3. With the rotor set to this option, use the single-finger swipe up or down to navigate through all the headings found on the screen.
   - Note: (1) the single-finger swipe left or right is still available at any time. (2) Setting the rotor to other options will allow the single-finger swipe up or down to navigate to that respective option.

**Android TalkBack Menu (formerly the Local Context Menu) setup**
1. Tap once with three fingers to access the TalkBack Menu.
2. Actions and Links are the most common options to use, but others such as Activate Links, Expand Accordions, Navigate through a Carousel, and Delete Elements are available too.

### Testing for enlarged text

- Go to Settings on your device and increase text size to 200%.
  - In iOS, with "Larger Accessibility Sizes" toggled ON, move the text-size slider to the 9th tick from the left.
  - In Android, move the slider to the last tick to the right, the "largest" or "Huge" setting.
- Ensure no text in the app is cut off, overlaps, truncates, or disappears.
- Ensure functionality works as expected; review accessibility acceptance criteria found on each component's individual reference page for testing steps.
  - In iOS, larger headings should adjust to Apple font size guidance.
- This does not apply to images of text or logos.
- Text in the top and bottom navigation bars **does not** increase in size.
- Text in a WebView on iOS does not increase in size.

**Developer note for large text**
- Ensure containers enlarge as text enlarges (view constraints are set correctly).
- Set labels to text-wrap.
- Specify font weight when needed.
- Specify font size and font scale when needed.
- Do not disable scrolling.
- Use preferred fonts for the platform when possible (designers will give developers the font styles and sizes).

## How to test with your Bluetooth external keyboard

Test with the Bluetooth external keyboard (without the screen reader turned on).

**Setting up your Bluetooth external keyboard**

Pair the external keyboard with your device as you would a computer.

For iOS setup only:
1. Open Settings
2. Choose Accessibility
3. Choose Keyboards
4. Toggle on "Full Keyboard Access"

**Navigating using your Bluetooth external keyboard**
- Turn OFF the screen reader when testing with the keyboard.
- Pair the external keyboard with the phone as you would a computer.
- iOS — turn on the Full Keyboard Access toggle in Accessibility settings or the keyboard will not work properly.
- The focus ring for keyboard, which is different from the screen reader focus, must be visible. Sometimes this is just a slight change or shadow.

**How to test with your Bluetooth external keyboard**
- Use the tab key OR arrow keys to navigate through the screen.
  - Other key combinations besides Tab, Shift-Tab, and Arrow keys are also acceptable — Ctrl+Tab is one example.
- Only interactive elements must be focused. If text or images are also focused, it is not a defect.
- Check that the element is functional with the keyboard.
  - iOS: only the Space bar activates the element.
  - Android: Space bar and/or Enter key activates an element.

**What is a defect with your Bluetooth external keyboard**
- Tab or arrow keys do not focus an interactive element.
- Tab order of elements is not logical or does not match the reading order.
- Enter and Space bar do not activate the element on Android.
- Space bar does not activate the element on iOS.

**Notes for using your Bluetooth external keyboard**
- Hybrid screens or WebViews in the native app can be buggy.
- Test the issue on a computer in a web page to verify that the bug is also on the web. If it is OK on the web, do not log it as a defect in the app — app screen readers trying to interpret web code do not always get it right.
- Known issue on both platforms: not being able to tab or arrow into the main part of a hybrid screen in a native app.

## How to test for links

**Navigate through the screen with iOS**
- Links can be activated through the Rotor (twist with two fingers to access the Rotor).
- Or double-tap to activate when they are focused on the screen.

**Navigate through the screen with Android**
- Links can be activated from the TalkBack menu (tap once with three fingers to access the menu).
- Or double-tap to activate when they are focused on the screen.

**Navigate through the screen using one of the following keys to reach all links**
- The `tab` and `shift tab` keys.
- The `arrow` keys.
- The `Ctrl+tab` and `Ctrl+shift tab` keys.
- Ensure links can be activated with the `space` key on iOS, and for Android, `enter` key and `space` key both work separately.

**Checklist**
- [ ] Each link receives focus and has a visible focus indicator — the indicator is slightly different for the screen reader and keyboard.
  - Pass: the control gets focus (e.g. a real focusable native control). Fail: a non-focusable view-like element never receives focus.
- [ ] All links have clear labels and all graphical controls have accurate programmatic names and roles.
  - Pass example: "See details for the iPhone 14". Fail example: "Get additional info on the specs for iPhone 14" (vague, inconsistent with visible label).
- [ ] Screen readers accurately announce any link state that is conveyed visually.
  - Disabled link — iOS: "Learn more, dimmed, link". Android: "Learn more, disabled".

## What's the difference between a link and a button

- If it opens a browser (i.e. outside the app), it's a **link**. A link can look like a big shiny button but it must be coded as a link.
- If the user stays within the app, it's a **button**. A button can look like a link, but it must be coded as a button.

**Checklist**
- [ ] Controls are announced correctly as links based on their function and purpose, regardless of visual design.
  - Note: the role of "link" communicates to screen reader users that they will be navigated outside the app once they interact with it.
  - Pass: upon activation, the user is navigated to a browser, outside the app. Fail: upon activation, the user stays within the app (this should be coded as a button instead).

## Flutter

Flutter apps are tested with the same OS-level screen readers (VoiceOver on iOS, TalkBack on Android) and the same external-keyboard testing procedure described above, since Flutter has no separate assistive-technology stack of its own — it renders through each platform's native accessibility tree. There is nothing platform-unique to add here beyond what's already documented for iOS and Android above; apply the same gesture, keyboard, and text-resize testing steps to a Flutter app as you would to any other native iOS or Android app.

## Related WCAG

- 2.4.4 Link Purpose (In Context)
- 2.5.3 Label in Name
- 3.2.4 Consistent Identification
- 4.1.2 Name, Role, Value

---
Source: https://github.com/tmobile/magentaA11y/blob/main/public/content/documentation/how-to-test/test-type/native-apps.md
