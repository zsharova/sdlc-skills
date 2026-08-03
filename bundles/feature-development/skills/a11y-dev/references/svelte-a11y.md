# Svelte / SvelteKit A11y Patterns

## ESLint — install first

```bash
npm install --save-dev eslint-plugin-svelte-a11y
```

`eslint.config.js`:
```js
import svelteA11y from 'eslint-plugin-svelte-a11y';

export default [
  {
    plugins: { 'svelte-a11y': svelteA11y },
    rules: {
      ...svelteA11y.configs.recommended.rules,
    },
  },
];
```

`.eslintrc.json` (legacy):
```json
{
  "plugins": ["svelte-a11y"],
  "extends": ["plugin:svelte-a11y/recommended"]
}
```

---

## ARIA attribute binding

```svelte
<!-- CORRECT: reactive ARIA bindings -->
<button aria-expanded={isOpen} aria-controls={menuId}>
  Menu
</button>

<!-- CORRECT: aria-live region -->
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

<!-- CORRECT: conditional aria-label -->
<button aria-label={isPlaying ? 'Pause' : 'Play'}>
  <Icon name={isPlaying ? 'pause' : 'play'} aria-hidden="true" />
</button>
```

---

## Focus management — Svelte action pattern

Svelte actions (`use:`) are the idiomatic way to manage focus:

```svelte
<!-- focusTrap action -->
<script>
  function focusTrap(node) {
    const focusable = node.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const first = focusable[0];
    const last = focusable[focusable.length - 1];

    function handleKeydown(e) {
      if (e.key !== 'Tab') return;
      if (e.shiftKey) {
        if (document.activeElement === first) {
          e.preventDefault();
          last.focus();
        }
      } else {
        if (document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      }
    }

    node.addEventListener('keydown', handleKeydown);
    first?.focus();

    return {
      destroy() {
        node.removeEventListener('keydown', handleKeydown);
      }
    };
  }
</script>

{#if isOpen}
  <div role="dialog" aria-modal="true" aria-labelledby="dialog-title" use:focusTrap>
    <h2 id="dialog-title">Confirm action</h2>
    <p>Are you sure?</p>
    <button on:click={confirm}>Confirm</button>
    <button on:click={close}>Cancel</button>
  </div>
{/if}
```

### Return focus on close

```svelte
<script>
  let triggerEl;
  let isOpen = false;

  function open() {
    triggerEl = document.activeElement;
    isOpen = true;
  }

  function close() {
    isOpen = false;
    // Return focus to trigger after DOM update
    tick().then(() => triggerEl?.focus());
  }
</script>

<button on:click={open}>Open dialog</button>
```

---

## SvelteKit — route change announcements

SvelteKit does not announce route changes to screen readers by default.
Add a route announcer to `+layout.svelte`:

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import { afterNavigate } from '$app/navigation';
  import { page } from '$app/stores';

  let announcement = '';

  afterNavigate(() => {
    // Announce the new page title after navigation
    announcement = '';
    tick().then(() => {
      announcement = $page.data.title
        ? `Navigated to ${$page.data.title}`
        : document.title;
    });
  });
</script>

<div
  role="status"
  aria-live="polite"
  aria-atomic="true"
  class="sr-only"
>
  {announcement}
</div>

<slot />
```

Set page titles in each `+page.server.ts` or `+page.ts`:

```ts
// src/routes/dashboard/+page.ts
export function load() {
  return { title: 'Dashboard' };
}
```

---

## Skip link

```svelte
<!-- src/routes/+layout.svelte — first element in DOM -->
<a href="#main-content" class="skip-link">Skip to main content</a>

<main id="main-content" tabindex="-1">
  <slot />
</main>

<style>
  .skip-link {
    position: absolute;
    left: -9999px;
    top: auto;
    width: 1px;
    height: 1px;
    overflow: hidden;
  }
  .skip-link:focus {
    position: static;
    width: auto;
    height: auto;
  }
</style>
```

---

## Visually hidden utility

```svelte
<!-- $lib/components/VisuallyHidden.svelte -->
<span class="sr-only"><slot /></span>

<style>
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border-width: 0;
  }
</style>
```

Usage:
```svelte
<button on:click={close}>
  <Icon name="x" aria-hidden="true" />
  <VisuallyHidden>Close dialog</VisuallyHidden>
</button>
```

---

## Icon-only buttons

```svelte
<!-- Always provide an accessible name -->
<button on:click={close} aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

---

## Form error patterns

```svelte
<script>
  let errors = {};
</script>

<div>
  <label for="email">Email address</label>
  <input
    id="email"
    type="email"
    autocomplete="email"
    aria-invalid={!!errors.email}
    aria-describedby={errors.email ? 'email-error' : undefined}
  />
  {#if errors.email}
    <p id="email-error" role="alert">
      {errors.email}
    </p>
  {/if}
</div>
```

---

## Reactive document title (SvelteKit)

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import { page } from '$app/stores';
  $: document.title = $page.data.title
    ? `${$page.data.title} — My App`
    : 'My App';
</script>
```

Or use `svelte:head`:

```svelte
<!-- src/routes/dashboard/+page.svelte -->
<svelte:head>
  <title>Dashboard — My App</title>
</svelte:head>
```

---

## Reduced motion

```svelte
<script>
  import { prefersReducedMotion } from '$lib/stores/motion';
</script>

<div
  class="animated"
  class:no-motion={$prefersReducedMotion}
>
  Content
</div>

<style>
  .animated { transition: transform 300ms ease; }
  .no-motion { transition: none; }

  @media (prefers-reduced-motion: reduce) {
    .animated { transition: none; }
  }
</style>
```

```ts
// $lib/stores/motion.ts
import { readable } from 'svelte/store';

export const prefersReducedMotion = readable(false, (set) => {
  if (typeof window === 'undefined') return;
  const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
  set(mq.matches);
  const handler = (e: MediaQueryListEvent) => set(e.matches);
  mq.addEventListener('change', handler);
  return () => mq.removeEventListener('change', handler);
});
```

---

## svelte-axe — dev-time a11y checking

```bash
npm install --save-dev axe-core
```

```ts
// src/lib/a11y.ts — dev only
if (import.meta.env.DEV) {
  import('axe-core').then(({ default: axe }) => {
    axe.run().then(({ violations }) => {
      if (violations.length) {
        console.group('axe a11y violations');
        violations.forEach(v => console.warn(v));
        console.groupEnd();
      }
    });
  });
}
```

Or use Playwright + axe for automated testing:

```ts
// e2e/a11y.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('homepage has no a11y violations', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();
  expect(results.violations).toEqual([]);
});
```

---

## html-validate (plain HTML projects)

For static HTML or server-rendered templates without a JS framework:

```bash
npm install --save-dev html-validate
```

`.htmlvalidate.json`:
```json
{
  "extends": ["html-validate:recommended"],
  "rules": {
    "aria-label-misuse": "error",
    "no-redundant-role": "error",
    "prefer-native-element": "warn",
    "wcag/h30": "error",
    "wcag/h32": "error",
    "wcag/h36": "error",
    "wcag/h37": "error",
    "wcag/h67": "error",
    "wcag/h71": "error"
  }
}
```

```bash
# Run against changed HTML files
npx html-validate "src/**/*.html"
```
