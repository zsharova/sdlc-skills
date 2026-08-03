# WCAG 2.2 AA Checklist

> **Spec reference:** this file follows the W3C **WCAG 2.2** Recommendation
> (October 2023), https://www.w3.org/TR/WCAG22/ — the canonical source for
> success criterion numbers, names, and conformance levels below. Where this
> file and another reference disagree on a level, WCAG 2.2 itself is the
> tiebreaker.

A practical pre-merge checklist organized by WCAG 2.2 success criterion,
covering **every level A/AA criterion developers control** — not just the 9
new-in-2.2 ones. For a deep implementation dive on those 9 new criteria
specifically (2.4.11, 2.4.12, 2.4.13, 2.5.7, 2.5.8, 3.2.6, 3.3.7, 3.3.8,
3.3.9), see [`wcag22-new-criteria.md`](./wcag22-new-criteria.md) — this file
is the broader full-checklist companion to that deep dive.

## Perceivable

- **1.1.1 Non-text Content** — Every `<img>` has `alt` (empty for decorative). Icons in buttons have `aria-hidden="true"` and the button has `aria-label`.
- **1.3.1 Info and Relationships** — Form fields associated with `<label>` via `htmlFor`. Headings used hierarchically. Lists are `<ul>`/`<ol>`/`<dl>`. Tables use `<th>` with `scope`.
- **1.3.2 Meaningful Sequence** — DOM order matches visual order. No CSS reordering that changes meaning.
- **1.3.3 Sensory Characteristics** — No "click the green button" instructions; reference by label.
- **1.3.4 Orientation** — Layout works in portrait and landscape.
- **1.3.5 Identify Input Purpose** — Use `autocomplete` attributes (`autocomplete="email"`, `autocomplete="current-password"`).
- **1.4.1 Use of Color** — State (error, required, selected) is conveyed by more than color (icon, text).
- **1.4.3 Contrast (Minimum) [AA]** — Normal text ≥4.5:1; large text (≥18.66px regular / ≥24px bold) ≥3:1.
- **1.4.4 Resize Text** — Text scales to 200% without loss of content.
- **1.4.5 Images of Text** — Use real text, not images of text (rare exceptions: logos).
- **1.4.10 Reflow [AA]** — At 320 CSS px width, content reflows; no horizontal scroll for vertical content.
- **1.4.11 Non-text Contrast [AA]** — UI components and graphical objects ≥3:1 against adjacent colors. Includes focus rings.
- **1.4.12 Text Spacing [AA]** — App survives line-height 1.5×, paragraph spacing 2×, letter-spacing 0.12×, word-spacing 0.16×.
- **1.4.13 Content on Hover or Focus [AA]** — Hover-revealed content is dismissible (Esc), hoverable, persistent until trigger removes it.

## Operable

- **2.1.1 Keyboard** — Every interactive element reachable and operable via keyboard.
- **2.1.2 No Keyboard Trap** — Focus can move away from any element with the keyboard alone.
- **2.1.4 Character Key Shortcuts** — Single-key shortcuts (e.g. typing "j" navigates) can be turned off, remapped, or only fire on focus.
- **2.4.1 Bypass Blocks** — Provide a "Skip to main content" link or `<main>` landmark.
- **2.4.2 Page Titled** — `<title>` describes the page.
- **2.4.3 Focus Order** — Tab order matches reading order.
- **2.4.4 Link Purpose (In Context)** — Link text describes the destination ("View report" not "Click here").
- **2.4.5 Multiple Ways [AA]** — Provide more than one way to find a page (nav + search, or sitemap).
- **2.4.6 Headings and Labels [AA]** — Headings and labels describe their topic.
- **2.4.7 Focus Visible [AA]** — Keyboard focus is visible.
- **2.4.11 Focus Not Obscured (Minimum) [AA, NEW in 2.2]** — Focused element not entirely hidden by sticky headers/footers.
- **2.4.12 Focus Not Obscured (Enhanced) [AAA]** — Focused element fully visible.
- **2.5.1 Pointer Gestures** — Multi-point gestures (pinch, swipe) have single-point alternatives.
- **2.5.2 Pointer Cancellation** — Click fires on `pointerup`, not `pointerdown` (default browser behavior).
- **2.5.3 Label in Name** — Visible label is part of the accessible name.
- **2.5.4 Motion Actuation** — Functions triggered by device motion (shake, tilt) have UI alternatives.
- **2.5.7 Dragging Movements [AA, NEW in 2.2]** — Drag actions have non-drag alternatives (e.g. up/down buttons next to drag handles).
- **2.5.8 Target Size (Minimum) [AA, NEW in 2.2]** — Interactive targets ≥24×24 CSS px (or have ≥24 px spacing).

## Understandable

- **3.1.1 Language of Page** — `<html lang="en">` (or appropriate code).
- **3.2.1 On Focus** — Focus does not trigger context change (no auto-form-submit on focus).
- **3.2.2 On Input** — Changing a field value does not trigger context change unless warned.
- **3.2.6 Consistent Help [A, NEW in 2.2]** — Help mechanisms (chat, contact, FAQ) are in the same place across pages.
- **3.3.1 Error Identification** — Errors identified in text and associated with the field via `aria-describedby`.
- **3.3.2 Labels or Instructions** — Form fields have labels or instructions.
- **3.3.3 Error Suggestion [AA]** — Error messages suggest a correction when possible.
- **3.3.4 Error Prevention (Legal/Financial) [AA]** — Reversible, checked, or confirmed before submission for high-stakes actions.
- **3.3.7 Redundant Entry [A, NEW in 2.2]** — Don't ask for the same info twice in a process.
- **3.3.8 Accessible Authentication (Minimum) [AA, NEW in 2.2]** — No cognitive function tests (e.g. "remember this code") without alternatives.

## Robust

- **4.1.2 Name, Role, Value** — Every UI component exposes a name, role, and current value to assistive tech.
- **4.1.3 Status Messages [AA]** — Status updates announced via `aria-live` regions or `role="status"` / `role="alert"`.

## WCAG AAA criteria — aspirational but impactful

AAA criteria are not required for general WCAG 2.2 conformance, but several
are **highly valuable** for products targeting users with disabilities. Know
these when auditing or bidding on government/healthcare projects.

| SC | Level | Summary | Key detail |
| --- | --- | --- | --- |
| **1.2.6** Sign Language | AAA | Prerecorded video includes a sign language interpreter track | Human interpreter embedded or as secondary stream |
| **1.4.6** Contrast (Enhanced) | AAA | Normal text ≥ **7:1**; large text ≥ **4.5:1** | AA is 4.5:1 / 3:1; double down for low-vision users |
| **1.4.7** Low/No Background Audio | AAA | Background audio ≥ **20 dB lower** than speech, or off-able | Applies to auto-playing audio with speech content |
| **2.1.3** Keyboard (No Exception) | AAA | Every function keyboard-operable, **no path-dependent exemption** | AA (2.1.1) exempts freehand drawing; AAA removes that exemption |
| **2.2.3** No Timing | AAA | **No time limits** except real-time events (live auction) | AA allows 20× extension; AAA requires no limit at all |
| **2.3.2** Three Flashes | AAA | No flashing at all beyond threshold (even below 3 Hz) | 2.3.1 (AA) allows threshold; 2.3.2 disallows period |
| **2.4.9** Link Purpose (Link Only) | AAA | Each link's purpose described by its **own accessible name alone** — no context reliance | AA allows surrounding context; AAA requires self-describing links |
| **2.4.10** Section Headings | AAA | Use headings to **organize** sections of content | Adds to 2.4.6 (headings must be good *when present*) by requiring them for sectioned content |
| **2.4.12** Focus Not Obscured (Enhanced) | AAA | Focused element **fully** visible (not just partially) | AA 2.4.11 allows partial visibility |
| **2.4.13** Focus Appearance | AAA | Focus indicator area ≥ 2 CSS px perimeter **area formula**; 3:1 contrast between states | Area = perimeter × 2 (see formula below) |
| **2.5.5** Target Size (Enhanced) | AAA | Targets ≥ **44 × 44 CSS px**, no offset exception | AA 2.5.8 allows 24 px with spacing exception |

### WCAG 2.4.13 Focus Appearance — Area Formula

```
Required area ≥ sum of (perimeter of component bounding box) × 2 CSS px
```

For a 100 × 40 px button:
- Perimeter = 2 × (100 + 40) = 280 px
- Required focus indicator area = 280 × 2 = **560 sq px**

A 2 px solid outline creates: perimeter × 2 = exactly the minimum. A 1 px outline fails.
Use `outline: 2px` or `box-shadow` with offset ≥ 2 px.

The 3:1 contrast is **between focused and unfocused states** (not just vs background).

### WCAG 1.4.7 Audio Control / Background Audio

Two criteria interact:

- **1.4.2 Audio Control (AA)**: Auto-playing audio > 3 s → provide pause/stop/volume control as **first** element.
- **1.4.7 Low/No Background Audio (AAA)**: Speech audio has no background, or background is −20 dB below speech, or user can turn off background.

Always implement 1.4.2. Add a mute button before `<main>` to satisfy both.

## Quick checklist (5-minute pre-merge)

- [ ] Tab through the new page/component — every interactive reachable, focus visible.
- [ ] Press Enter/Space on each interactive — does the action fire?
- [ ] Inspect with `axe-core` (browser devtools or `@axe-core/playwright` — see [`testing-a11y.md`](./testing-a11y.md)).
- [ ] Verify text contrast (devtools color picker, or design tokens).
- [ ] Read with VoiceOver or NVDA on the happy path — does it make sense?
- [ ] Headings hierarchical and labeled? Landmarks (`<main>`, `<nav>`, etc.)?

## How to test — WCAG 2.2 new criteria

| SC | Quick test |
| --- | --- |
| **1.3.5** Identify Input Purpose | Inspect inputs: `autocomplete="email"`, `current-password`, `street-address`, etc. |
| **1.4.10** Reflow | DevTools → 320px width; no horizontal scroll for vertical reading content. |
| **1.4.11** Non-text Contrast | Measure focus ring and icon buttons vs adjacent bg — ≥3:1. |
| **1.4.12** Text Spacing | Bookmarklet or extension: apply 1.5× line-height, 2× paragraph spacing — content must not clip. |
| **2.1.4** Character Key Shortcuts | List global `keydown` shortcuts; confirm off-by-default, focus-scoped, or remappable. |
| **2.4.11** Focus Not Obscured | Tab through page with sticky header; focused control must not be fully hidden. |
| **2.4.13** Content on Hover or Focus | Open tooltip; move pointer onto tooltip; press Esc — must dismiss; must not require hover path that disappears. |
| **2.5.7** Dragging Movements | Every drag handle has buttons/menu alternative to reorder or move. |
| **2.5.8** Target Size | Measure clickable area — ≥24×24 CSS px or spacing equivalent. |
| **3.2.6** Consistent Help | Compare help/chat/FAQ placement across two routes — same relative location. |
| **3.3.7** Redundant Entry | Walk multi-step form — same field must not be re-entered without reason. |
| **3.3.8** Accessible Authentication | No puzzle-only or "remember this picture" login without OTP/WebAuthn alternative. |

See [`../SKILL.md`](../SKILL.md) for the skill overview.
