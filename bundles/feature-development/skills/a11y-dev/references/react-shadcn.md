# React + shadcn/ui + Radix UI A11y Patterns

Sourced from: https://github.com/michelve/accessibility SKILL.md
Stack: React 19 · shadcn/ui · Tailwind CSS v4 · Radix UI (radix-ui v1.4.3)

---

## Built-in Accessibility — What Radix Handles Automatically

shadcn/ui components are accessible by default — built on Radix UI primitives that
handle ARIA roles, keyboard navigation, and focus management. Do not re-implement
what these components already provide.

| Component | What Radix handles |
|-----------|-------------------|
| `Dialog` | `role="dialog"`, `aria-modal`, focus trap (`FocusScope`), `Escape` to close, return focus on close |
| `AlertDialog` | `role="alertdialog"`, focus trap, prevents accidental `Escape` dismissal |
| `DropdownMenu` | `role="menu"` / `menuitem`, roving tabindex, Arrow keys, `Escape` to close |
| `Select` | `role="listbox"` / `option`, Arrow keys, type-ahead search |
| `Tabs` | `role="tablist"` / `tab` / `tabpanel`, Arrow Left/Right navigation |
| `Checkbox` | `role="checkbox"`, `Space` to toggle, `aria-checked` |
| `Tooltip` | `role="tooltip"`, hoverable (pointer can move over it), dismissible with `Escape`, persistent |
| `Popover` | Focus-managed popup, `Escape` to close, returns focus to trigger |
| `Accordion` | `aria-expanded`, Arrow keys for panel navigation |
| `Slider` | `role="slider"`, `aria-valuenow` / `min` / `max`, Arrow key adjustments |
| `Switch` | `role="switch"`, `aria-checked`, `Space`/`Enter` to toggle |

---

## Critical Rules

### Never modify shadcn components directly
Components live in `src/client/components/ui/` and must never be modified.
Customisation goes in wrapper components that compose them.

### Always include Dialog.Close
`Dialog.Content` traps focus correctly, but if `Dialog.Close` is missing, users
cannot dismiss via keyboard. This is a Critical gap.

```tsx
// ✅ Correct
<Dialog>
  <DialogContent>
    <DialogTitle>Confirm deletion</DialogTitle>
    <p>This cannot be undone.</p>
    <DialogClose asChild>
      <Button variant="outline">Cancel</Button>
    </DialogClose>
    <Button variant="destructive">Delete</Button>
  </DialogContent>
</Dialog>
```

### Focus ring — never removed by styling
Custom styles frequently remove Radix's default focus ring. Always restore it explicitly:

```tsx
// ✅ Standard focus ring — follow button.tsx pattern
className="focus-visible:ring-[3px] focus-visible:ring-ring/50 focus-visible:outline-none"

// ✅ With Focus Not Obscured fix (WCAG 2.4.11 AA) for sticky header
className="focus-visible:ring-[3px] focus-visible:ring-ring/50 focus-visible:outline-none focus-visible:scroll-mt-20"

// ❌ Wrong — triggers on mouse too, and targets may remove ring accidentally
className="focus:outline-none"
```

### Never use focus: alone for ring styles
`focus:` triggers on mouse click. `focus-visible:` triggers on keyboard only.

---

## Accessibility Utilities

| Utility | Import / Usage | Purpose |
|---------|---------------|---------|
| `FocusScope` | `import { FocusScope } from 'radix-ui'` | Focus trapping for custom modals (already used by Dialog) |
| `sr-only` | `className="sr-only"` | Visually hidden, screen-reader visible |
| `focus-visible:` | Tailwind variant | Focus ring on keyboard only |
| `motion-reduce:` | `motion-reduce:transition-none` | Respects `prefers-reduced-motion` |
| `aria-*:` | `aria-invalid:border-destructive` | Tailwind variant for ARIA state styling |
| `scroll-mt-20` | `focus-visible:scroll-mt-20` | Scroll margin for 2.4.11 AA |
| `min-h-6 min-w-6` | 24×24px minimum target (2.5.8 AA) | |
| `min-h-[44px] min-w-[44px]` | 44×44px recommended target (2.5.5 AAA) | |

---

## Icon-Only Buttons

Always provide an accessible name — either `aria-label` or `sr-only` span:

```tsx
// Option A — aria-label
<button aria-label="Close modal">
  <XIcon aria-hidden="true" className="h-4 w-4" />
</button>

// Option B — sr-only span (both are valid)
<button>
  <XIcon aria-hidden="true" className="h-4 w-4" />
  <span className="sr-only">Close modal</span>
</button>
```

---

## Screen Reader Only Text

```tsx
// Visually hide text but keep it in the a11y tree
<span className="sr-only">Additional context for screen readers</span>

// Reverse — make sr-only content visible
<span className="not-sr-only">Now visible</span>
```

---

## Install Components

```bash
npx shadcn@latest add dialog dropdown-menu select tabs tooltip popover
```

Components appear in `src/client/components/ui/`. Do not edit them directly.

---

## ESLint — install for static analysis

```bash
npm install -D eslint-plugin-jsx-a11y
```

```js
// eslint.config.js
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  jsxA11y.flatConfigs.recommended,
  // ... rest of config
];
```

---

## Color Contrast Reference (Project Design Tokens)

Semantic token pairs from `src/client/index.css` (`oklch` values):

| Pair | Token classes | Status |
|------|---------------|--------|
| Body text | `text-foreground bg-background` | ~16:1 ✔ |
| Primary action | `bg-primary text-primary-foreground` | passes ✔ |
| Destructive/error | `bg-destructive text-destructive-foreground` | passes ✔ |
| Secondary surface | `bg-secondary text-secondary-foreground` | passes ✔ |
| Muted text | `text-muted-foreground` | ⚠ verify at your font size |

Verify custom combinations at: https://webaim.org/resources/contrastchecker/
or https://apcacontrast.com/
