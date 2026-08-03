# Accessibility Anti-patterns

Common mistakes shown as BAD/GOOD code pairs, each with the concrete failure
mode. Use this when you see one of these shapes in a diff and want the
canonical fix, or as a checklist while writing new UI.

## `<div onClick>` masquerading as a button

> 🔴 **Critical** — A `<div onClick>` is not focusable, not announced as a button by screen readers, and doesn't respond to Enter/Space; keyboard and assistive-tech users cannot activate it at all.

```tsx
// BAD — not focusable, no keyboard handler, no role
<div className="btn" onClick={save}>Save</div>

// GOOD
<button type="button" onClick={save}>Save</button>
```

## Outline removed without replacement

> 🔴 **Critical** — `outline: none` without a replacement makes it impossible for keyboard users to see which element has focus, causing WCAG 2.4.7 (Focus Visible) and 2.4.11 (Focus Appearance) failures.

```css
/* BAD — keyboard users have no visible focus */
button:focus { outline: none; }

/* GOOD — replace, don't remove */
button:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 2px; }
```

## Placeholder as the only label

> 🔴 **Critical** — A placeholder disappears as soon as the user starts typing, leaving them with no reminder of what the field requires; screen readers may not announce it at all, failing WCAG 1.3.1 and 3.3.2.

```tsx
// BAD — placeholder disappears on focus; low contrast; not announced
<input type="email" placeholder="Email" />

// GOOD
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

## Color-only state

> 🔴 **Critical** — Conveying error or status by color alone fails WCAG 1.4.1 (Use of Color), making the state invisible to colorblind users and causing form validation errors to go unnoticed.

```tsx
// BAD — colorblind users can't tell error from normal
<input className={hasError ? 'border-red-500' : 'border-gray-300'} />

// GOOD — pair color with text and icon
<input
  className={hasError ? 'border-danger' : 'border-border'}
  aria-invalid={hasError}
  aria-describedby={hasError ? 'email-error' : undefined}
/>
{hasError && <p id="email-error" role="alert"><WarnIcon /> {errorMessage}</p>}
```

## ARIA when HTML would do

> 🟡 **Warning** — A custom `role="button"` `<div>` requires manually implementing all keyboard behavior that `<button>` provides for free; omitting any key handler (e.g. Space) silently breaks keyboard users.

```tsx
// BAD
<div role="button" tabIndex={0} onClick={save} onKeyDown={(e) => e.key === 'Enter' && save()}>Save</div>

// GOOD
<button type="button" onClick={save}>Save</button>
```

## `aria-hidden` on a focusable element

> 🔴 **Critical** — An element that is both focusable and `aria-hidden` can receive keyboard focus but announces nothing to screen readers, creating a ghost focus state that disorients assistive-tech users.

```tsx
// BAD — element is focusable but invisible to screen readers
<button aria-hidden="true">Save</button>

// GOOD — either remove from tab order or remove aria-hidden
<button type="button" onClick={save}>Save</button>
// OR (decorative-only)
<svg aria-hidden="true" focusable="false">...</svg>
```

SVG icons inside buttons should be `aria-hidden` with `focusable="false"` so the button's text label isn't drowned out.

## Skipped heading levels

> 🟡 **Warning** — Skipping heading levels (`<h1>` to `<h4>`) breaks the document outline that screen reader users rely on to navigate a page, causing them to miss entire content sections.

```tsx
// BAD
<h1>Settings</h1>
<h4>Profile</h4>      {/* skipped h2, h3 */}

// GOOD
<h1>Settings</h1>
<h2>Profile</h2>
```

## Modal that doesn't trap focus

> 🔴 **Critical** — Focus leaking out of a modal causes keyboard users to interact with hidden page content behind it, and screen readers announce that hidden content as if it were accessible.

Tab continues into the page behind the modal — keyboard users get lost. Always trap focus inside the dialog and restore it on close (see [`focus-management.md`](./focus-management.md)).

```tsx
// React — use FocusScope (radix-ui), focus-trap-react, or Radix Dialog
import { FocusScope } from "radix-ui";
<FocusScope loop trapped><Dialog /></FocusScope>
```

```html
<!-- Angular — cdkTrapFocus (@angular/cdk/a11y), or MatDialog which wraps it automatically -->
<div cdkTrapFocus cdkTrapFocusAutoCapture>
  <!-- dialog content -->
</div>
```

```ts
// Vue — useFocusTrap (@vueuse/integrations/useFocusTrap)
import { useFocusTrap } from '@vueuse/integrations/useFocusTrap'
const modalRef = ref<HTMLElement>()
const { activate, deactivate } = useFocusTrap(modalRef)
```

```svelte
<!-- Svelte — a use: action (see svelte-a11y.md for the full focusTrap action) -->
<div role="dialog" aria-modal="true" use:focusTrap>
  <!-- dialog content -->
</div>
```

Prefer your framework's accessible-primitives library (Radix/shadcn Dialog,
Angular Material `mat-dialog`, Headless UI) over any of the above hand-rolled
traps when one is already a dependency — see
[`component-library-risks.md`](./component-library-risks.md).

## Positive `tabindex`

> 🟡 **Warning** — Positive `tabindex` values override the natural DOM tab order, creating a navigation sequence that is completely different from the visual layout and confuses keyboard users.

```tsx
// BAD — fights with the natural tab order
<input tabIndex={1} />
<input tabIndex={2} />

// GOOD — DOM order is the tab order
<input />
<input />
```

## Missing accessible name on icon button

> 🔴 **Critical** — An icon-only button without `aria-label` is announced as just "button" by screen readers with no indication of its purpose, making the action completely unusable without vision.

```tsx
// BAD — screen reader announces "button"
<button><CloseIcon /></button>

// GOOD
<button aria-label="Close"><CloseIcon aria-hidden="true" /></button>
```

## `alt` text on every image, including decorative

> 🟢 **Note** — Describing every decorative image forces screen reader users to sit through irrelevant announcements (e.g. "decorative pattern") on every page load, degrading their browsing experience without conveying meaning.

```tsx
// BAD — screen reader reads "decorative pattern" pointlessly
<img src="/dots.svg" alt="Decorative pattern" />

// GOOD — empty alt; screen reader skips
<img src="/dots.svg" alt="" />
```

## Live region with `aria-live="assertive"` for non-urgent updates

> 🟡 **Warning** — `aria-live="assertive"` interrupts whatever the screen reader is currently announcing; using it for routine status messages (saves, confirmations) breaks the user's reading flow unexpectedly.

```tsx
// BAD — interrupts user mid-sentence for a save confirmation
<div aria-live="assertive">Saved</div>

// GOOD
<div aria-live="polite">Saved</div>
```

## `role="alert"` for status messages

> 🟡 **Warning** — `role="alert"` is `aria-live="assertive"` + `aria-atomic="true"` and carries the same interrupt risk; overusing it for non-error statuses disrupts assistive-tech users constantly.

`role="alert"` is `aria-live="assertive"` + `aria-atomic="true"`. Same warning — reserve for genuine errors, not status.

See [`../SKILL.md`](../SKILL.md) for the skill overview.
