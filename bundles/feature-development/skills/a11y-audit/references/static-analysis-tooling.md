# Static Code Analysis & Custom-Rule Options for Accessibility, by Framework

This is a reference doc for the sibling skill `a11y-audit`
(`../SKILL.md`) — it isn't a standalone workflow, just the detailed
"which linter, why, and what its limits are" material that skill pulls from
at two points in its review flow: when it bootstraps missing a11y linters
for a detected stack (the skill's "Linter bootstrap" step), and when it
audits a project's UI component library for known accessibility risks (the
skill's "Component Library Audit" step). If you landed here directly: read
`../SKILL.md` first for how this fits into the overall review.

Free/open-source tooling only — no paid tool is ever proposed as an install
by `a11y-audit`; Deque's axe DevTools Linter is documented below purely as
context for what a paid tool additionally buys you, never as a recommendation.

## TL;DR

- **Every major framework has a mature, dedicated a11y linter**:
  `eslint-plugin-jsx-a11y` (React), `@angular-eslint/eslint-plugin-template`
  (Angular, ships an `accessibility` preset), `eslint-plugin-vuejs-accessibility`
  (Vue/Nuxt), the Svelte **compiler**'s built-in `a11y_*` warnings surfaced via
  `eslint-plugin-svelte`, and `@html-eslint/eslint-plugin` (plain HTML). All
  support custom rules via ESLint's standard rule API plus framework-specific
  AST utilities.
- **No open-source linter understands component-library APIs at the prop
  level.** They resolve a custom component to a plain HTML element (via a
  `components`/mapping setting) and lint it as if it were that element — they
  do not read a component's TypeScript prop types or know that, say, a Radix
  `Dialog.Content` needs an accessible title. The only tool that gets closer
  is the **commercial** Deque axe DevTools Linter, whose `global-components`
  mapping simulates emitted markup from component props; not proposed here.
- **PrimeNg/PrimeVue ship no ESLint rules of their own** — their
  "accessibility support" is runtime ARIA configuration (an `aria`
  locale/translation object, directive-bound `aria-label`s), not static
  analysis. Statically checking their component props requires either a
  custom rule you write, or the commercial tool above.
- Static analysis is universally incomplete — pair it with runtime axe-core
  testing (unit tests via `vitest-axe`/`jest-axe`, Storybook a11y addon).
  E2e axe testing (Playwright/Cypress) is **out of scope for this skill** —
  see `bundles/test-automation`'s future `a11y-test-automation` skill.

## Per-framework linter choice

What `a11y-audit`'s linter-bootstrap step installs, by detected stack:

### React (incl. shadcn/Radix)
- **`eslint-plugin-jsx-a11y`** — the de facto standard. Ships `recommended`
  and `strict` shareable configs (flat config via
  `jsxA11y.flatConfigs.recommended`). Depends on `aria-query`,
  `axobject-query`, `jsx-ast-utils`.
- Install: `npm i -D eslint-plugin-jsx-a11y`

**Component-mapping mechanism** (`settings["jsx-a11y"]`):
```jsonc
{
  "settings": {
    "jsx-a11y": {
      "polymorphicPropName": "as",
      "components": { "CustomButton": "button", "MyButton": "button" },
      "attributes": { "for": ["htmlFor", "for"] }
    }
  }
}
```
- `components` maps a custom component name to a DOM element so it's linted
  as that element.
- `polymorphicPropName` (+ `polymorphicAllowList`) handles `as`-style
  polymorphism (`<Box as="h3">` linted as `h3`) — directly relevant to
  Radix/shadcn's `asChild` pattern.

**Limit for shadcn/Radix specifically**: shadcn copies Radix primitive source
into the repo, so accessibility behavior lives in Radix internals. jsx-a11y
can only check it once mapped to an element, and even then cannot verify
prop contracts like "Radix `Dialog` needs a `DialogTitle`" or "`asChild`
child is focusable" — the plugin's own docs note static analysis cannot
evaluate values placed in props before runtime. Combine with manual keyboard
testing and runtime axe.

### Angular (incl. CDK/Material, PrimeNg)
- **`@angular-eslint/eslint-plugin-template`** — official. Ships an
  **`accessibility` preset** (`plugin:@angular-eslint/template/accessibility`,
  included by default in recent recommended flat configs). Rules ported from
  jsx-a11y: `alt-text`, `click-events-have-key-events`, `elements-content`,
  `interactive-supports-focus`, `label-has-associated-control`,
  `mouse-events-have-key-events`, `no-autofocus`, `no-distracting-elements`,
  `no-positive-tabindex`, `role-has-required-aria`, `table-scope`,
  `valid-aria`.
- Install: `ng add @angular-eslint/schematics`

**Custom rule authoring** (well-documented via `@angular-eslint/utils` +
`@angular-eslint/test-utils`): template rules use
`getTemplateParserServices(context)` and CSS-like selectors against the
Angular template AST:
```ts
import type { TmplAstElement } from '@angular-eslint/bundled-angular-compiler';
import { getTemplateParserServices } from '@angular-eslint/utils';
import { ESLintUtils } from '@typescript-eslint/utils';

export const rule = ESLintUtils.RuleCreator.withoutDocs({
  meta: { type: 'suggestion', messages: { requireAriaLabel: 'Missing aria-label' }, schema: [] },
  defaultOptions: [],
  create(context) {
    const parserServices = getTemplateParserServices(context);
    return {
      'Element[name="p-calendar"]'(node: TmplAstElement) {
        const hasAriaLabel = node.attributes.some(a => a.name === 'ariaLabel' || a.name === 'aria-label');
        if (!hasAriaLabel) {
          context.report({ loc: parserServices.convertNodeSourceSpanToLoc(node.sourceSpan), messageId: 'requireAriaLabel' });
        }
      },
    };
  },
});
```
This is the pattern to reach for when a **PrimeNg** component needs
prop-level enforcement — the template AST exposes element name + attributes,
so a selector like `Element[name="p-calendar"]` can check for a missing
`ariaLabel`. What the built-in rules do **not** do: read a component's
TypeScript `@Input()` definitions, check attributes/roles set via the `host`
section of `@Component` or `@HostBinding`, or evaluate bound values
(`[attr.aria-label]="x"`). `label-has-associated-control` supports
`labelComponents`/`controlComponents` options to extend recognition to
custom components.

**PrimeNg's actual extension surface** is runtime, not static: a global
`aria` locale/translation object plus directive-bound ARIA
(`<p-calendar [ariaLabel]="...">`). Whether that prop is actually set is a
static-analysis question you answer with a custom rule (above) — PrimeNg
itself ships no ESLint rules. Angular Material/CDK relies on the CDK a11y
module (`FocusMonitor`, `LiveAnnouncer`, `cdkTrapFocus`) at runtime, not
static lint.

### Vue / Nuxt (incl. Vuetify, PrimeVue)
- **`eslint-plugin-vuejs-accessibility`** — the maintained standard (not
  `eslint-plugin-vue-a11y`, which is stale — avoid it). ESLint v9 flat-config
  ready (`pluginVueA11y.configs["flat/recommended"]`). 24+ rules: alt-text,
  anchor-has-content, click-events-have-key-events, form-control-has-label,
  heading-has-content, role-has-required-aria-props, media-has-caption, etc.
- Install: `npm i -D eslint-plugin-vuejs-accessibility`

**Component-mapping**: several rules accept component-name options
(label/control component lists), analogous to jsx-a11y's `components`. For
**Vuetify**, `eslint-plugin-vuetify`/`eslint-config-vuetify` exist but target
deprecation/migration (`no-deprecated-components` etc.), **not** accessibility
prop checking. There is no PrimeVue- or Vuetify-specific a11y ESLint plugin —
same custom-rule-authoring path as Angular applies here, via
`vue-eslint-parser`'s template AST.

**Runtime overlay**: `vue-axe` (dev-time console warnings, analogous to
`@axe-core/react`) — see the "Runtime axe overlay" step in `a11y-audit`.

### Svelte / SvelteKit
A11y checks live in the **compiler**, not ESLint — ~20+ warnings at compile
time (`a11y_missing_attribute`, `a11y_click_events_have_key_events`,
`a11y_no_static_element_interactions`, `a11y_autofocus`,
`a11y_role_has_required_aria_props`, `a11y_distracting_elements`, etc.),
mostly ported from `eslint-plugin-jsx-a11y`. Suppress individually with
`<!-- svelte-ignore a11y_autofocus -->`.

- **`eslint-plugin-svelte`** surfaces these via its `svelte/valid-compile`
  rule (reads `onwarn`/`warningFilter` from `svelte.config.js`). There is no
  separate comprehensive Svelte a11y ESLint plugin.
- **`svelte-check`** (CLI) reports the same compiler warnings for CI.
- Install: `npm i -D eslint-plugin-svelte`

**Limitation**: compiler a11y warnings are not extensible the way ESLint
rules are — you can only ignore them, not author new ones. The compiler
doesn't evaluate dynamic attribute values, doesn't understand component
boundaries, and event-directive-based rules have documented false
positives/negatives. No component-library-API-aware checking exists for
Svelte.

### Plain HTML
- **`@html-eslint/eslint-plugin`** — ESLint's new language-agnostic system
  (`language: "html/html"`), 70+ rules covering best-practices, style, SEO,
  and accessibility.
  ```js
  import { defineConfig } from "eslint/config";
  import html from "@html-eslint/eslint-plugin";
  export default defineConfig([
    { files: ["**/*.html"], plugins: { html }, language: "html/html",
      rules: { /* a11y rules */ } }
  ]);
  ```
- Alternative: **`html-validate`** — configurable, widely paired with Angular
  projects.
- Install: `npm i -D @html-eslint/eslint-plugin` or `npm i -D html-validate`

### CSS (any stack)
- **`stylelint-a11y`** (paired with `stylelint` if not already present) —
  catches `outline: none` without a focus replacement, insufficient contrast
  in CSS custom properties, `display:none` on focusable elements, missing
  `forced-colors`/high-contrast media queries.
- Install: `npm i -D stylelint stylelint-a11y` (or just `stylelint-a11y` if
  `stylelint` is already present).

## The "understands component-library APIs" question — direct answer

**Open-source ESLint plugins: no true API-awareness.** The universal
mechanism is element-mapping (jsx-a11y's `components`, Angular/Vue rules'
`labelComponents`/similar options, `polymorphicPropName` for `as`/`asChild`).
This maps `<MyButton>` → `button` and lints it as if it were a native
`<button>` — it does not read the component's prop/type API. You can
*approximate* prop-level checking with a custom rule (patterns above, fully
supported in React/Angular/Vue), but you author that logic yourself; no
open-source linter ships built-in knowledge of shadcn/Radix/PrimeNg/PrimeVue/
Vuetify/Angular Material component APIs.

**The one tool with true API-awareness is commercial**: Deque axe DevTools
Linter's `global-components` config maps a component's props onto a native
element (e.g. `DqButton: { element: button, attributes: [role, aria-*,
{action: type}, {label: <text>}] }`) and runs axe rules on the simulated
result; it ships preconfigured definitions for `@mui/material`,
`@deque/cauldron-react`, and `react-native` (no preconfigured definition
exists for shadcn, Radix, PrimeNg, PrimeVue, Vuetify, or Angular Material —
you'd author `global-components` mappings yourself even with this tool).
**Not proposed by `a11y-audit`** — mentioned here only so the "why can't the
free linter catch this" question has a documented, honest answer instead of
a workaround that doesn't exist.

## Staged recommendation (what `a11y-audit`'s linter bootstrap actually proposes)

1. **Baseline** — the framework's native a11y ruleset (table above), always
   proposed first if missing.
2. **Close the component-API gap** — for design-system components in the
   diff, either add them to the linter's element-mapping setting, or author
   a small custom rule (patterns above) when true prop-contract enforcement
   is needed. `a11y-audit`'s Component Library Audit step surfaces this per
   detected library; it does not auto-write custom rules, only recommends
   the pattern.
3. **Always pair with runtime/unit testing** — static analysis catches a
   minority of WCAG issues on its own; combine with `vitest-axe`/`jest-axe`
   (`a11y-audit`'s unit-test-generation step) and, where relevant, a runtime
   dev overlay (its runtime-axe-overlay step).
4. **Never propose the paid tool** — if a team decides they need
   component-API-aware static analysis badly enough to buy it, that's a
   decision for them to make outside this skill, not a recommendation this
   skill issues.

## Caveats

- Static analysis cannot evaluate dynamic/bound prop values (e.g. Angular
  `[attr.aria-label]="expr"`, a variable passed as a React prop) — stated by
  jsx-a11y, Angular ESLint, and the Svelte compiler docs alike. Green lint is
  necessary, not sufficient.
- `eslint-plugin-vue-a11y` (no "js") is stale — use
  `eslint-plugin-vuejs-accessibility`.
- Svelte's compiler a11y warnings are not extensible — you can only ignore
  them, and they have documented blind spots (dynamic values, component
  boundaries, event-directive false positives/negatives).
