# Vue / Nuxt A11y Patterns

## ESLint — install first

```bash
npm install --save-dev eslint-plugin-vuejs-accessibility
```

`.eslintrc.json`:
```json
{
  "plugins": ["vuejs-accessibility"],
  "extends": ["plugin:vuejs-accessibility/recommended"]
}
```

## Dynamic ARIA bindings

```html
<!-- CORRECT: bind as prop, not string -->
<button :aria-expanded="isOpen" :aria-controls="menuId">Menu</button>

<!-- CORRECT: aria-live region -->
<div aria-live="polite" aria-atomic="true">
  {{ statusMessage }}
</div>

<!-- WRONG: string literal -->
<button aria-expanded="true">Menu</button>
```

## Focus management — useFocusTrap (VueUse)

```typescript
import { useFocusTrap } from '@vueuse/integrations/useFocusTrap'

const modalRef = ref<HTMLElement>()
const { activate, deactivate } = useFocusTrap(modalRef)

watch(isOpen, (open) => {
  if (open) {
    nextTick(() => activate())
  } else {
    deactivate()
    triggerRef.value?.focus() // return focus to trigger
  }
})
```

## Nuxt — route change announcements

```typescript
// plugins/a11y-announcer.client.ts
export default defineNuxtPlugin(() => {
  const router = useRouter()
  router.afterEach(() => {
    nextTick(() => {
      const announcer = document.getElementById('route-announcer')
      if (announcer) {
        announcer.textContent = document.title
      }
    })
  })
})
```

```html
<!-- layouts/default.vue -->
<div
  id="route-announcer"
  role="status"
  aria-live="polite"
  aria-atomic="true"
  class="sr-only"
/>
```

## Visually hidden utility

```css
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
```

## Icon-only buttons

```html
<button @click="close" :aria-label="'Close ' + dialogTitle">
  <IconX aria-hidden="true" />
</button>
```

## vue-axe — dev-time a11y checking

```bash
npm install --save-dev vue-axe
```

```typescript
// main.ts — dev only
if (process.env.NODE_ENV !== 'production') {
  const VueAxe = await import('vue-axe')
  app.use(VueAxe.default)
}
```
