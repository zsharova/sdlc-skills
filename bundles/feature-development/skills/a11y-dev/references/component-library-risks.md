# Component Library A11y Risks

## React ecosystem

### MUI (Material UI)
- `TextField` requires `label` prop — without it, no accessible name on input.
- `IconButton` has no visible label — always add `aria-label`.
- `Dialog` — accessible by default if `aria-labelledby` points to `DialogTitle`.
- `Select` — accessible if `labelId` prop matches a `InputLabel` id.
- `Tooltip` — announced on focus, but only if child is a focusable element.
  Wrapping a `<span>` around a `<button>` breaks this — use `<button>` directly.
- `DataGrid` — virtualised. Requires `aria-rowcount` + `aria-rowindex` for
  correct screen reader row count. Not set by default in community edition.

### Radix UI / shadcn
- Keyboard and ARIA behaviour built-in for all primitives.
- **Risk**: visual styling often removes default focus ring. Always add
  `:focus-visible` styles explicitly.
- `Dialog.Content` traps focus correctly. Ensure `Dialog.Close` always present.
- `Select` — uses ARIA listbox pattern. Verify announcements with NVDA.

### Headless UI (Tailwind)
- ARIA roles provided. Keyboard navigation provided.
- **Risk**: same as Radix — focus ring must be added manually via CSS.

### React Aria (Adobe)
- Most accessible React library available. Use when accessibility is critical.
- Handles screen reader virtualisation edge cases other libraries miss.

## Angular ecosystem

### Angular Material
- Generally accessible. Most components fully WCAG 2.1 AA compliant.
- `mat-form-field` requires `mat-label` or `aria-label` on the inner input.
- `mat-select` — accessible but announces differently on NVDA vs JAWS. Test both.
- `mat-dialog` — uses `cdkTrapFocus` correctly. Ensure `mat-dialog-title` present.
- `mat-table` — accessible for simple tables. Virtual scroll requires manual
  `aria-rowcount` management.

### Angular CDK
- `LiveAnnouncer` — correct tool for dynamic announcements. Prefer over
  `aria-live` regions managed manually.
- `cdkTrapFocus` — use in all modal-like components.
- `FocusMonitor` — use to detect keyboard vs mouse focus for `:focus-visible`
  equivalent behaviour.

## Vue ecosystem

### Vuetify
- Generally accessible. `v-text-field` requires `label` prop.
- `v-dialog` — accessible if `aria-labelledby` set. Focus trap built-in.
- `v-data-table` — same virtual scroll risk as MUI DataGrid.

### PrimeVue
- Mixed record. Test every component with NVDA before shipping.
- `Dialog` accessible. `DataTable` — verify row count announcements.
- `AutoComplete` — uses combobox pattern but may announce incorrectly on NVDA.

### Headless UI (Vue)
- Same as React version — ARIA provided, focus ring must be added via CSS.

## Known inaccessible patterns (flag immediately)

- Old Bootstrap 3/4 modals — do not use `aria-hidden` on backdrop correctly.
- jQuery UI widgets — largely inaccessible. No modern ARIA support.
- Custom `<div>` dropdowns without ARIA — always flag as Critical.
- `contenteditable` without `role="textbox"` and `aria-multiline`.
- `<canvas>` without fallback text content or `aria-label`.
