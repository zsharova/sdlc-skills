# Testing Accessibility

## Browser DevTools (zero setup)

- **Chrome DevTools** → Elements panel → Accessibility tab: inspect the a11y tree, roles, and properties for any element
- **axe DevTools** browser extension (free tier): full page audit with issue explanations
- **WAVE** browser extension: visual overlay of a11y issues

## Playwright + @axe-core/playwright (e2e - recommended)

Playwright is already configured in this project. Add `@axe-core/playwright` as an optional dev dependency for automated audits:

```bash
npm install -D @axe-core/playwright
```

```ts
// e2e/a11y.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("home page has no accessibility violations", async ({ page }) => {
    await page.goto("/");
    const results = await new AxeBuilder({ page }).analyze();
    expect(results.violations).toEqual([]);
});

// Scope to a specific component
test("modal is accessible", async ({ page }) => {
    await page.goto("/");
    await page.getByRole("button", { name: "Open" }).click();
    const results = await new AxeBuilder({ page }).include('[role="dialog"]').analyze();
    expect(results.violations).toEqual([]);
});
```

## Vitest + vitest-axe (unit tests - optional)

For component-level a11y checks, install `vitest-axe` if needed:

```bash
npm install -D vitest-axe
```

```ts
import { test, expect } from 'vitest';
import { render } from '@testing-library/react';
import { axe } from 'vitest-axe';

test('component has no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**React 18 concurrent mode + `jest-axe`** — state updates must be wrapped in `act()`:

```ts
import { render, act } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe } from 'jest-axe';

it('form with validation is accessible', async () => {
  const { container } = render(<LoginForm />);

  // Trigger async validation inside act()
  await act(async () => {
    await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
  });

  // Check after async validation has settled
  expect(await axe(container)).toHaveNoViolations();
});
```

`@testing-library/react` 14+ with `userEvent` handles `act()` automatically for user interactions. For manual state changes, wrap in `act(async () => {...})`.

---

## Screen Reader Quick Reference

### Essential Shortcuts

| Screen Reader | OS | Navigation keys |
| ------------- | -- | --------------- |
| **NVDA** | Windows | `Insert+F7` element list · `H` headings · `D` landmarks · `B` buttons · `F` form fields · `T` tables |
| **JAWS** | Windows | `Insert+F6` headings list · `Insert+F3` links list · `R` rows in table · `F` forms mode |
| **VoiceOver** | macOS/iOS | `VO+U` rotor · `VO+Cmd+H` heading · `VO+Cmd+L` links · `VO+Space` activate |
| **TalkBack** | Android | Swipe left/right navigate · Double-tap activate · Swipe up then right for Reading Controls |
| **Narrator** | Windows | `Caps+F6` headings · `Caps+F7` links · `Caps+Enter` activate · `Caps+Space` action |

### Priority Test Scenarios

For each screen reader, verify at minimum:

1. Skip link works and moves focus to `#main-content`
2. Page title announced on load/route change
3. All images have meaningful alternative text
4. Form fields announced with label + error
5. Dialogs trap focus and announce `role="dialog"` + accessible name
6. Live regions (role="status" / role="alert") announced without focus move
7. Custom widgets (combobox, tree, carousel) follow APG key model

---

## CI/CD Automated Accessibility Checks

**Minimal single-file snippet** (illustrative — e2e a11y CI wiring itself is
out of scope for the `a11y-audit` skill; see `bundles/test-automation`'s
future `a11y-test-automation` skill for that):

```yaml
# .github/workflows/accessibility.yml
name: Accessibility
on: [push, pull_request]

jobs:
  axe:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npx serve dist &
      - run: npx playwright test e2e/a11y.spec.ts
```

**Tag axe tests so they run independently:**

```ts
test("home page passes axe audit @a11y", async ({ page }) => { … });
```

```bash
# Run only a11y tests locally
npx playwright test --grep @a11y
```

---

## Axe Rule Customization

Disable specific rules only when you have a documented justification:

```ts
const results = await new AxeBuilder({ page })
  .withRules(["color-contrast", "keyboard-navigation"]) // test only these
  .analyze();

// OR disable a rule with documented reason
const results = await new AxeBuilder({ page })
  .disableRules(["color-contrast"]) // ⚠️ document why in a comment
  .analyze();
```

**Useful axe rule tags:**

| Tag | Covers |
| --- | ------- |
| `wcag2a` | WCAG 2.1 Level A |
| `wcag2aa` | WCAG 2.1 Level AA |
| `wcag22aa` | WCAG 2.2 Level AA (new criteria) |
| `best-practice` | Non-normative best practices |
| `experimental` | Rules under development |

```ts
// Test only WCAG 2.2 AA rules
const results = await new AxeBuilder({ page })
  .withTags(["wcag2a", "wcag2aa", "wcag22aa"])
  .analyze();
```

### `.exclude()` vs `violations.filter()` — when to use which

| Approach | What it does | Use for |
| --- | --- | --- |
| `.exclude('#selector')` | Axe does **not scan** these elements at all | Third-party iframes, legacy embeds you cannot fix |
| `violations.filter(v => ...)` | Axe scans, but you filter specific violations out of the result | Known false positives within your own code |

```ts
// Third-party iframe — exclude entirely
const results = await new AxeBuilder({ page })
  .exclude('#stripe-iframe')
  .analyze();

// Known false positive in own code — filter with explanation
const filtered = results.violations.filter(v =>
  // TODO: JIRA-1234 — brand-color violation tracked separately via token linting
  !(v.id === 'color-contrast' && v.nodes.every(n => n.target.includes('.brand-cta')))
);
expect(filtered).toEqual([]);
```

Always add a comment with the ticket reference — silently discarding violations is a process failure.

### Shadow DOM scanning

Standard axe-core scans do not pierce shadow DOM boundaries without configuration:

```ts
// @axe-core/playwright — scan shadow DOM components
const results = await new AxeBuilder({ page })
  .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
  .options({ iframes: true })  // scan iframes
  .analyze();
```

For web components with shadow roots, use axe-core 4.7+ (improved native shadow DOM support). If violations are missed inside web components, scan the shadow host directly or use the `shadowDomComponent` option.

### Custom axe rules for design token contrast

```ts
import axe from 'axe-core';

axe.configure({
  rules: [{
    id: 'brand-primary-contrast',
    selector: '.text-brand-primary',
    tags: ['wcag2aa', 'brand'],
    all: ['color-contrast'],
    metadata: {
      description: 'Brand primary text must meet 4.5:1 contrast on white',
      help: 'Use design token brand-primary (verified at 7:1 in light mode)',
    },
  }],
});
```

**Better for token-level validation**: use a design token linting step (style-dictionary contrast plugin, Token Studio) at build time — this catches oklch/P3 color issues before they reach the browser.

---

## Manual Testing Checklist (Abbreviated)

**Quick smoke test (5 minutes per page):**

- [ ] Tab through the page — no keyboard traps, logical order
- [ ] All interactive elements reachable by keyboard and have visible focus
- [ ] Skip link present and functional
- [ ] No content only distinguishable by color
- [ ] All images have alt text (inspect with DevTools or WAVE)
- [ ] Zoom to 400% at 1280px — no horizontal scroll, no content loss
- [ ] Test with screen reader: announce headings, form labels, errors

---

## Audit Cadence

| Frequency | Activity |
| --------- | -------- |
| Every PR | Axe CI check (automated) |
| Weekly | Dev self-test with keyboard + WAVE |
| Monthly | Screen reader test (NVDA + VoiceOver) |
| Quarterly | Full POUR manual audit + VPAT update |

---

## VPAT & Accessibility Statement

A Voluntary Product Accessibility Template (VPAT) documents how a product
conforms to accessibility standards. Required for enterprise customers and
government procurement.

**Quick facts:**

- Use **VPAT 2.5 WCAG Edition** for web products
- Conformance levels: Supports / Partially Supports / Does Not Support / Not Applicable
- Update the VPAT after each major release or quarterly audit
- Public accessibility statement required for WCAG self-declaration

---

## Testing Pyramid

```text
         /‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
        /  Manual + SR    \   ← Quarterly full audit + VPAT
       /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
      /  e2e axe (Playwright) \  ← Every PR via CI
     /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
    /  Unit vitest-axe (Vitest)  \  ← During development
   /________________________________\
  /  Static: TypeScript + ESLint a11y \  ← Continuous (editor)
 /______________________________________\
```

Install `eslint-plugin-jsx-a11y` for continuous static analysis in the editor:

```bash
npm install -D eslint-plugin-jsx-a11y
```

```js
// eslint.config.js
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  jsxA11y.flatConfigs.recommended,
  // … rest of your config
];
```

**Recommended rules to keep enabled** (baseline configs sometimes disable these):

| Rule | Catches |
| --- | --- |
| `alt-text` | `<img>` without meaningful `alt` |
| `anchor-is-valid` | `<a>` without `href` used as button |
| `click-events-have-key-events` | `onClick` on non-interactive elements without keyboard equivalent |
| `no-static-element-interactions` | `<div onClick>` without role + keyboard |
| `label-has-associated-control` | Inputs without labels |
| `aria-props` / `aria-proptypes` | Invalid ARIA attribute names/values |
| `role-has-required-aria-props` | `role="menu"` missing required children, etc. |
| `tabindex-no-positive` | `tabIndex={1}` anti-pattern |

ESLint runs at author time; axe runs in the browser — use **both**.
