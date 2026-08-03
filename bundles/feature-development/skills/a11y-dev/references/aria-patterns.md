# ARIA Patterns Reference

Common ARIA patterns for draft_v0 components (React 19 + shadcn/ui).

## Contents

- [Button Pattern](#button-pattern)
- [Input Pattern](#input-pattern)
- [Checkbox Pattern](#checkbox-pattern)
- [Radio Group Pattern](#radio-group-pattern)
- [Switch/Toggle Pattern](#switchtoggle-pattern)
- [Dialog/Modal Pattern](#dialogmodal-pattern)
- [Alert Pattern](#alert-pattern)
- [Menu Pattern](#menu-pattern)
- [Tabs Pattern](#tabs-pattern)
- [Accordion Pattern](#accordion-pattern)
- [Tooltip Pattern](#tooltip-pattern)
- [Progress Pattern](#progress-pattern)
- [Live Region Pattern](#live-region-pattern)
- [Table Pattern](#table-pattern)
- [Combobox (autocomplete)](#combobox-autocomplete)
- [Slider](#slider)
- [Landmarks](#landmarks)
- [Listbox vs Menu](#listbox-vs-menu)
- [Advanced ARIA attributes](#advanced-aria-attributes)
- [Live regions — advanced patterns](#live-regions--advanced-patterns)
- [Complex table headers](#complex-table-headers)
- [Focus management — virtual lists](#focus-management--virtual-lists)
- [Shadow DOM and focus](#shadow-dom-and-focus)
- [Rich text editor](#rich-text-editor)
- [Keyboard Navigation Reference](#keyboard-navigation-reference)
- [Screen Reader Announcements](#screen-reader-announcements)

---

## Button Pattern

```tsx
<button
    type="button"
    aria-pressed={isPressed} // For toggle buttons
    aria-expanded={isExpanded} // For buttons that expand/collapse
    aria-haspopup="true" // For menu buttons
    aria-controls="menu-id" // ID of controlled element
    aria-disabled={isDisabled} // When visually disabled but focusable
    disabled={isDisabled} // When truly disabled
>
    Button Label
</button>
```

## Input Pattern

```tsx
<div>
    <label htmlFor="input-id" id="label-id">
        Label Text
    </label>
    <input
        id="input-id"
        type="text"
        aria-labelledby="label-id"
        aria-describedby="help-id error-id"
        aria-required={isRequired}
        aria-invalid={hasError}
        aria-errormessage="error-id"
    />
    <span id="help-id">Help text</span>
    {hasError && (
        <span id="error-id" role="alert">
            Error message
        </span>
    )}
</div>
```

## Checkbox Pattern

```tsx
<label>
    <input
        type="checkbox"
        checked={isChecked}
        aria-checked={isChecked} // For custom checkboxes
        aria-describedby="desc-id"
    />
    <span>Checkbox Label</span>
</label>
```

## Radio Group Pattern

```tsx
<fieldset>
    <legend>Radio Group Label</legend>
    <div role="radiogroup" aria-labelledby="legend-id">
        <label>
            <input
                type="radio"
                name="group-name"
                value="option1"
                aria-checked={selectedValue === "option1"}
            />
            Option 1
        </label>
        <label>
            <input
                type="radio"
                name="group-name"
                value="option2"
                aria-checked={selectedValue === "option2"}
            />
            Option 2
        </label>
    </div>
</fieldset>
```

## Switch/Toggle Pattern

```tsx
<button
    type="button"
    role="switch"
    aria-checked={isOn}
    aria-label="Toggle feature"
    onClick={toggle}
>
    <span aria-hidden="true">{isOn ? "ON" : "OFF"}</span>
</button>
```

## Dialog/Modal Pattern

```tsx
<div
    role="dialog"
    aria-modal="true"
    aria-labelledby="dialog-title"
    aria-describedby="dialog-description"
>
    <h2 id="dialog-title">Dialog Title</h2>
    <p id="dialog-description">Dialog description</p>

    <button type="button" aria-label="Close dialog">
        ×
    </button>

    <div>{/* Dialog content */}</div>

    <footer>
        <button type="button">Cancel</button>
        <button type="submit">Confirm</button>
    </footer>
</div>
```

## Alert Pattern

```tsx
// Assertive alert (interrupts screen reader)
<div role="alert" aria-live="assertive">
  Error: Something went wrong
</div>

// Polite notification (waits for pause)
<div role="status" aria-live="polite">
  Changes saved successfully
</div>
```

## Menu Pattern

```tsx
<div>
    <button type="button" aria-haspopup="true" aria-expanded={isOpen} aria-controls="menu-id">
        Menu
    </button>

    <ul id="menu-id" role="menu" aria-labelledby="menu-button-id" hidden={!isOpen}>
        <li role="menuitem" tabIndex={-1}>
            Option 1
        </li>
        <li role="menuitem" tabIndex={-1}>
            Option 2
        </li>
        <li role="separator" aria-hidden="true" />
        <li role="menuitem" tabIndex={-1}>
            Option 3
        </li>
    </ul>
</div>
```

## Tabs Pattern

```tsx
<div>
    <div role="tablist" aria-label="Tab navigation">
        <button
            role="tab"
            aria-selected={activeTab === 0}
            aria-controls="panel-0"
            id="tab-0"
            tabIndex={activeTab === 0 ? 0 : -1}
        >
            Tab 1
        </button>
        <button
            role="tab"
            aria-selected={activeTab === 1}
            aria-controls="panel-1"
            id="tab-1"
            tabIndex={activeTab === 1 ? 0 : -1}
        >
            Tab 2
        </button>
    </div>

    <div role="tabpanel" id="panel-0" aria-labelledby="tab-0" hidden={activeTab !== 0} tabIndex={0}>
        Panel 1 content
    </div>
    <div role="tabpanel" id="panel-1" aria-labelledby="tab-1" hidden={activeTab !== 1} tabIndex={0}>
        Panel 2 content
    </div>
</div>
```

## Accordion Pattern

```tsx
<div>
    <h3>
        <button
            type="button"
            aria-expanded={isExpanded}
            aria-controls="section-content"
            id="section-header"
        >
            Section Title
        </button>
    </h3>
    <div id="section-content" role="region" aria-labelledby="section-header" hidden={!isExpanded}>
        Section content
    </div>
</div>
```

## Tooltip Pattern

```tsx
<div>
    <button
        type="button"
        aria-describedby="tooltip-id"
        onMouseEnter={showTooltip}
        onMouseLeave={hideTooltip}
        onFocus={showTooltip}
        onBlur={hideTooltip}
    >
        Hover me
    </button>

    <div id="tooltip-id" role="tooltip" hidden={!isVisible}>
        Tooltip content
    </div>
</div>
```

## Progress Pattern

```tsx
// Determinate progress
<div
  role="progressbar"
  aria-valuenow={progress}
  aria-valuemin={0}
  aria-valuemax={100}
  aria-label="Upload progress"
>
  {progress}%
</div>

// Indeterminate progress
<div
  role="progressbar"
  aria-label="Loading"
  aria-busy="true"
>
  <span className="spinner" aria-hidden="true" />
</div>
```

## Live Region Pattern

```tsx
// For dynamic content updates
<div
    aria-live="polite" // or "assertive" for urgent
    aria-atomic="true" // Announce entire region
    aria-relevant="additions text" // What changes to announce
>
    {/* Dynamic content */}
</div>
```

## Table Pattern

```tsx
<table aria-labelledby="table-caption">
    <caption id="table-caption">Table Title</caption>
    <thead>
        <tr>
            <th scope="col">Column 1</th>
            <th scope="col">Column 2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">Row 1</th>
            <td>Data</td>
        </tr>
    </tbody>
</table>
```

## Combobox (autocomplete)

The most complex APG pattern. Use a library (Headless UI, React Aria, Downshift) — do not roll your own.

Required ARIA when you do roll your own:

```tsx
<input
  role="combobox"
  type="text"
  aria-expanded={open}
  aria-autocomplete="list"
  aria-controls="listbox-1"
  aria-activedescendant={activeId}
  value={value}
  onChange={...}
/>
{open && (
  <ul role="listbox" id="listbox-1">
    <li role="option" id="opt-1" aria-selected={selected === '1'}>Option 1</li>
  </ul>
)}
```

Keyboard: ArrowDown opens menu / next option, ArrowUp prev, Enter selects, Esc closes.

## Slider

```tsx
<input
  type="range"
  min={0}
  max={100}
  value={value}
  aria-valuemin={0}
  aria-valuemax={100}
  aria-valuenow={value}
  aria-valuetext={`${value} percent`}
  onChange={...}
/>
```

## Landmarks

Use semantic landmarks — do not duplicate:

```html
<header>...</header>
<nav aria-label="Main">...</nav>
<main id="main" tabIndex={-1}>...</main>
<aside aria-label="Filters">...</aside>
<footer>...</footer>
```

One `<main>` per page. Multiple `<nav>` elements need distinct `aria-label` values.

## Listbox vs Menu

- **Listbox** — selecting from a set of options (e.g. a custom select). Items have `role="option"`. Use `aria-activedescendant` on the combobox input.
- **Menu** — invoking commands (e.g. context menu). Items have `role="menuitem"`. **Home/End** jump to first/last item.

Pick the right one based on intent — never `role="menuitem"` inside a listbox.

## Advanced ARIA attributes

### `aria-roledescription` — override role's human-readable name

Use when a custom widget's role doesn't convey its purpose:

```tsx
// SR announces: "Fix login bug — Kanban card" instead of "Fix login bug — article"
<article
  aria-roledescription="Kanban card"
  aria-label="Fix login bug — high priority"
>
```

Rules: only use on widget/landmark roles (never on generic roles); element must have an accessible name; must be a meaningful replacement, not a workaround for a wrong role.

### `aria-details` — link to extended descriptions

Use for complex supplementary content (charts with data tables, code with documentation):

```tsx
// aria-describedby inlines short text; aria-details links to navigable content
<figure aria-label="Revenue chart" aria-details="revenue-table">
  <canvas .../>
</figure>
<table id="revenue-table">
  <caption>Full data for revenue chart</caption>
  ...
</table>
```

`aria-describedby` is for short descriptions (inlined in accessible name computation). `aria-details` is for extended, navigable supplementary content.

### `aria-keyshortcuts` — expose keyboard shortcuts to AT

```tsx
<div
  role="grid"
  aria-label="Data grid"
  aria-keyshortcuts="Control+f"  // format: "Modifier+Key"
>
```

Screen readers announce the shortcut when the element receives focus. Always also document shortcuts visually (help dialog, tooltip). Modifier-key shortcuts satisfy WCAG 2.1.4.

### `aria-errormessage` — semantic error association

```tsx
// Preferred over aria-describedby for validation errors:
<input
  aria-invalid={hasError ? 'true' : 'false'}
  aria-errormessage={hasError ? 'email-error' : undefined}
/>
<div id="email-error" role="alert">
  {hasError ? 'Enter a valid email address' : ''}
</div>
```

`aria-errormessage` is announced by SR only when `aria-invalid="true"`. The referenced element **must be visible** (not `display:none`). For older SR support (pre-2019), keep `aria-describedby` as fallback alongside `aria-errormessage`.

### `aria-busy` — async loading state

```tsx
<section aria-busy={isLoading} aria-label="Revenue card">
  {isLoading
    ? <span className="sr-only">Loading…</span>
    : <>{data}</>
  }
</section>
```

Set `aria-busy="true"` when content is loading; `aria-busy="false"` (or omit) when loaded. Add a live region to announce completion.

### `aria-rowcount` / `aria-rowindex` — virtual / large grids

When only a subset of rows is rendered (virtualized lists):

```tsx
<table
  role="grid"
  aria-rowcount={10000}      // total rows in the dataset
  aria-colcount={8}          // total columns
>
  {visibleRows.map((row, i) => (
    <tr
      role="row"
      aria-rowindex={row.absoluteIndex + 1}  // 1-based position in full dataset
      key={row.id}
    >
```

When total is unknown (infinite scroll): `aria-rowcount="-1"`. SR reads `aria-rowindex` instead of DOM row position.

### `role="application"` — disable SR virtual cursor

Turn off browse/virtual mode when all interaction is custom (code editors, spreadsheets):

```tsx
<div
  role="application"
  aria-label="Code editor"
  aria-describedby="editor-instructions"
>
  <span id="editor-instructions" className="sr-only">
    Press F1 for keyboard shortcut help. Press Tab to move to the next section.
  </span>
  ...
</div>
```

**Requirements alongside `role="application"`**:
1. All functionality keyboard-accessible.
2. Descriptive `aria-label` and user instructions.
3. An exit mechanism (Tab should leave the widget, or Escape).

Only use for widgets with a fully custom interaction model — never for content that benefits from browse-mode reading (cards, article lists, informational panels).

### `role="presentation"` / `role="none"` — remove table semantics from layout tables

```html
<!-- Layout table — remove grid semantics so SR doesn't announce "table 3 cols" -->
<table role="presentation">
  <tr><td>Logo</td><td>Nav</td><td>CTA</td></tr>
</table>
```

Better alternative: replace the layout table with CSS Grid/Flexbox. `role="presentation"` is a remediation technique.

## Live regions — advanced patterns

### Pre-insert the container before injecting content

```tsx
// In the initial render (before any updates):
<div role="status" aria-live="polite" aria-atomic="true" id="search-status" />

// When search completes, update content:
document.getElementById('search-status').textContent = `Showing ${count} results`;
```

**Never** conditionally render the live region with the content already inside — it won't announce.

### Live region inside a modal

When `aria-modal="true"` is active, some screen readers restrict live region monitoring to the dialog subtree. Ensure:
1. The live region is **inside the dialog container**.
2. It is pre-inserted empty when the dialog opens.

```tsx
<div role="dialog" aria-modal="true">
  <div role="status" aria-live="polite" id="dialog-status" />
  ...
</div>
```

### `aria-atomic` vs `aria-relevant`

- `aria-atomic="true"`: announce the entire container on any change (good for count summaries).
- `aria-atomic="false"` (default): announce only the changed nodes.
- `aria-relevant="additions text"` (default): announce added nodes and text changes.

For a search result count (`"Showing 12 results"`), use `aria-atomic="true"` to prevent announcing just `"12"` when the number changes.

### Toast / notification queue

> See also: `aria-attributes.md`'s `useAnnounce` hook (general-purpose announce) and
> `focus-management.md`'s `RouteAnnouncer` (route-change specific) — consolidate to one
> live-region utility per app instead of maintaining three.

```tsx
// Single persistent live region — never multiple simultaneous ones
const LiveRegion = () => (
  <div role="status" aria-live="polite" aria-atomic="true" id="toast-region" />
);

// Queue dispatcher — one toast at a time with ≥ 1 s delay
function showToast(message: string) {
  const el = document.getElementById('toast-region')!;
  el.textContent = '';
  setTimeout(() => { el.textContent = message; }, 50); // clear + re-insert for re-announcement
}
```

Multiple simultaneous live regions cause announcement race conditions across all screen readers.

## Complex table headers

For tables with merged cells (`colspan`/`rowspan`):

```html
<table>
  <thead>
    <tr>
      <th id="name" colspan="2">Name</th>
      <th id="dept">Department</th>
    </tr>
    <tr>
      <th id="fname" headers="name">First</th>
      <th id="lname" headers="name">Last</th>
      <th headers="dept">Division</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td headers="name fname">John</td>
      <td headers="name lname">Smith</td>
      <td headers="dept">Engineering</td>
    </tr>
  </tbody>
</table>
```

`scope="colgroup"` covers simple colspan headers; `id` + `headers` handles arbitrary complexity.

## Focus management — virtual lists

For virtualized / infinite-scrolling lists (items un-mount when off-screen):

```tsx
// Use aria-activedescendant on the container — items never directly receive DOM focus
<ul
  role="listbox"
  tabIndex={0}
  aria-activedescendant={activeId}
  onKeyDown={handleArrowKeys}
>
  {visibleItems.map(item => (
    <li
      role="option"
      id={`opt-${item.id}`}
      aria-selected={item.id === selectedId}
      tabIndex={-1}  // programmatic only
    >{item.label}</li>
  ))}
</ul>
```

When focus would land on an un-rendered item: scroll it into view, re-render it, then `.focus()`. Track the last focused item id across re-renders.

## Shadow DOM and focus

Focus traps using `querySelectorAll` **do not** pierce shadow DOM boundaries:

```ts
// Wrong — misses focusables inside web components:
const focusables = container.querySelectorAll('button, input, ...');

// Right — use the `inert` attribute (crosses shadow DOM boundaries automatically):
document.querySelector('main').inert = true;
// Or traverse shadow roots manually:
function getFocusables(root: Element): HTMLElement[] {
  const direct = Array.from(root.querySelectorAll<HTMLElement>('button, [tabindex="0"], ...'));
  const inShadow = Array.from(root.querySelectorAll('*'))
    .filter(el => el.shadowRoot)
    .flatMap(el => getFocusables(el.shadowRoot!));
  return [...direct, ...inShadow];
}
```

`inert` is the most robust cross-shadow-DOM focus containment: it works at the browser level and removes elements from tab order regardless of shadow DOM depth.

## Rich text editor

```tsx
// For a custom contenteditable rich text editor:
<div
  contentEditable="true"
  role="textbox"
  aria-multiline="true"
  aria-label="Document body"
  aria-describedby="editor-instructions"
>
</div>
```

For toolbar toggle buttons (Bold, Italic):

```tsx
<button
  type="button"
  role="button"
  aria-pressed={isBold}
  onClick={toggleBold}
>Bold</button>
```

**Recommendation**: use established libraries (Tiptap, Lexical, Slate) which handle accessibility — building a fully accessible rich text editor from scratch is extremely complex.

## Keyboard Navigation Reference

| Component | Key              | Action                |
| --------- | ---------------- | --------------------- |
| Button    | Enter, Space     | Activate              |
| Checkbox  | Space            | Toggle                |
| Radio     | Arrow Up/Down    | Select prev/next      |
| Menu      | Arrow Down       | Open menu / next item |
| Menu      | Arrow Up         | Previous item         |
| Menu      | Escape           | Close menu            |
| Tabs      | Arrow Left/Right | Switch tabs           |
| Dialog    | Escape           | Close dialog          |
| Dialog    | Tab              | Cycle focus within    |

## Screen Reader Announcements

```tsx
// Visually hidden but announced
<span className="visually-hidden">
  Screen reader only text
</span>

// Hide from screen readers
<span aria-hidden="true">
  Visual decoration only
</span>

// Announce on change
<div aria-live="polite" aria-atomic="true">
  {count} items selected
</div>
```
