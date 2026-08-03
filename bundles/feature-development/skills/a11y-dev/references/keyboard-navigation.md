# Keyboard Navigation

## Contents

- [Required Keys by Component Type](#required-keys-by-component-type)
- [useKeyPress Hook](#usekeypress-hook)
- [Inline Arrow Key Navigation](#inline-arrow-key-navigation)
- [Keyboard Patterns by APG Pattern](#keyboard-patterns-by-apg-pattern)
  - [Tree View](#tree-view)
  - [Carousel](#carousel)
  - [Toolbar](#toolbar)
  - [Combobox](#combobox)
  - [Data Grid](#data-grid)
- [Roving tabindex](#roving-tabindex)
- [Skip link](#skip-link)
- [Character key shortcuts (2.1.4)](#character-key-shortcuts-214)
- [Advanced keyboard patterns](#advanced-keyboard-patterns)
  - [Escape propagation hierarchy](#escape-propagation-hierarchy)
  - [F6 landmark region cycling](#f6-landmark-region-cycling)
  - [Printable character typeahead](#printable-character-typeahead-listbox-tree-grid)
  - [Multiselect listbox (Shift+Arrow)](#multiselect-listbox-shiftarrow)
- [Pitfalls](#pitfalls)
- [Pointer Cancellation (WCAG 2.5.2 AA)](#pointer-cancellation-wcag-252-aa)

---

## Required Keys by Component Type

| Component     | Keys                                                              |
| ------------- | ----------------------------------------------------------------- |
| Button        | `Enter`, `Space` to activate                                      |
| Link          | `Enter` to navigate                                               |
| Menu/Dropdown | `Arrow Up/Down` to navigate, `Enter` to select, `Escape` to close |
| Tabs          | `Arrow Left/Right` to switch, `Home/End` for first/last           |
| Modal         | `Tab` to cycle focus, `Escape` to close                           |
| Checkbox      | `Space` to toggle                                                 |
| Radio         | `Arrow Up/Down` to change selection                               |

## useKeyPress Hook

Create this hook at `src/client/hooks/useKeyPress.ts` (`@/hooks/useKeyPress`):

```ts
// src/client/hooks/useKeyPress.ts
import { useEffect } from "react";

export function useKeyPress(keys: string[], handler: (e: KeyboardEvent) => void) {
    useEffect(() => {
        const listener = (e: KeyboardEvent) => {
            if (keys.includes(e.key)) {
                handler(e);
            }
        };
        window.addEventListener("keydown", listener);
        return () => window.removeEventListener("keydown", listener);
    }, [keys, handler]);
}
```

Usage example:

```tsx
import { useKeyPress } from "@/hooks/useKeyPress";

function SearchModal() {
    const [isOpen, setIsOpen] = useState(false);

    // Cmd+K or Ctrl+K to open search
    useKeyPress(["k"], (e) => {
        if (e.metaKey || e.ctrlKey) {
            e.preventDefault();
            setIsOpen(true);
        }
    });

    // Escape to close
    useKeyPress(["Escape"], () => setIsOpen(false));

    return isOpen ? <SearchDialog onClose={() => setIsOpen(false)} /> : null;
}
```

## Inline Arrow Key Navigation

For single-use keyboard handlers, use an inline `onKeyDown` - no utility import needed:

```tsx
// Vertical navigation (menus, lists)
const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "ArrowUp") {
        e.preventDefault();
        focusPreviousItem();
    } else if (e.key === "ArrowDown") {
        e.preventDefault();
        focusNextItem();
    } else if (e.key === "Escape") {
        onClose?.();
    } else if (e.key === "Enter") {
        onActivate?.();
    }
};

// Horizontal navigation (tabs, toolbars)
const handleHorizontal = (e: React.KeyboardEvent) => {
    if (e.key === "ArrowLeft") selectPrevious();
    if (e.key === "ArrowRight") selectNext();
};
```

> **Note:** For complex components (menus, comboboxes, data grids), use the shadcn
> `DropdownMenu`, `Select`, or `Combobox` components instead - they handle roving
> tabindex and arrow key navigation automatically via Radix primitives.

---

## Keyboard Patterns by APG Pattern

### Tree View

| Key | Action |
| --- | ------ |
| `Arrow Down` | Move to next visible item (skips collapsed subtrees) |
| `Arrow Up` | Move to previous visible item |
| `Arrow Right` | If collapsed → expand; if expanded → focus first child |
| `Arrow Left` | If expanded → collapse; if child → move to parent; if root → no-op |
| `Enter` / `Space` | Select / activate the focused item |
| `Home` | Jump to first visible item |
| `End` | Jump to last visible item |
| Printable char | Type-ahead: jump to first matching visible item |

```tsx
function TreeItem({ item, level }: { item: TreeNode; level: number }) {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case "ArrowDown":
        e.preventDefault();
        focusNextVisible();
        break;
      case "ArrowUp":
        e.preventDefault();
        focusPrevVisible();
        break;
      case "ArrowRight":
        e.preventDefault();
        if (item.children?.length) {
          if (!item.expanded) expand(item.id);
          else focusFirstChild(item.id);
        }
        break;
      case "ArrowLeft":
        e.preventDefault();
        if (item.expanded) collapse(item.id);
        else if (level > 0) focusParent(item.id);
        break;
      case "Enter":
      case " ":
        e.preventDefault();
        select(item.id);
        break;
      case "Home":
        e.preventDefault();
        focusFirst();
        break;
      case "End":
        e.preventDefault();
        focusLast();
        break;
    }
  };

  return (
    <li
      role="treeitem"
      aria-level={level}
      aria-expanded={item.children?.length ? item.expanded : undefined}
      aria-selected={item.selected}
      tabIndex={item.focused ? 0 : -1}
      onKeyDown={handleKeyDown}
    >
      {item.label}
    </li>
  );
}
```

---

### Carousel

| Key | Action |
| --- | ------ |
| `Arrow Left` | Previous slide |
| `Arrow Right` | Next slide |
| `Home` | First slide |
| `End` | Last slide |
| `Enter` / `Space` | Activate focused slide link/control |

**Auto-rotation rules (WCAG 2.2.2):**

- MUST pause on any keyboard focus entering the carousel region
- MUST pause on pointer hover
- MUST NOT resume until focus/hover leaves AND the user has not activated Pause

```tsx
function Carousel({ slides }: { slides: Slide[] }) {
  const [current, setCurrent] = useState(0);
  const [paused, setPaused] = useState(false);
  const regionRef = useRef<HTMLDivElement>(null);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case "ArrowLeft":
        e.preventDefault();
        setCurrent((c) => (c - 1 + slides.length) % slides.length);
        break;
      case "ArrowRight":
        e.preventDefault();
        setCurrent((c) => (c + 1) % slides.length);
        break;
      case "Home":
        e.preventDefault();
        setCurrent(0);
        break;
      case "End":
        e.preventDefault();
        setCurrent(slides.length - 1);
        break;
    }
  };

  return (
    <section
      aria-roledescription="carousel"
      aria-label="Featured content"
      ref={regionRef}
      onFocus={() => setPaused(true)}
      onBlur={(e) => {
        if (!regionRef.current?.contains(e.relatedTarget as Node)) {
          setPaused(false);
        }
      }}
      onMouseEnter={() => setPaused(true)}
      onMouseLeave={() => setPaused(false)}
      onKeyDown={handleKeyDown}
    >
      {/* slides, controls, pause button */}
    </section>
  );
}
```

---

### Toolbar

| Key | Action |
| --- | ------ |
| `Arrow Left` / `Arrow Right` | Move between toolbar items (roving tabindex) |
| `Home` | First toolbar item |
| `End` | Last toolbar item |
| `Tab` | Exit the toolbar entirely (skip over remaining items) |

> Only **one** toolbar item has `tabIndex={0}` at a time. All others are `tabIndex={-1}`.
> `Tab` should always exit the toolbar — never navigate within it.

```tsx
function Toolbar({ actions }: { actions: Action[] }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const itemRefs = useRef<Array<HTMLButtonElement | null>>([]);

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    let next = index;
    if (e.key === "ArrowRight") next = (index + 1) % actions.length;
    else if (e.key === "ArrowLeft") next = (index - 1 + actions.length) % actions.length;
    else if (e.key === "Home") next = 0;
    else if (e.key === "End") next = actions.length - 1;
    else return;

    e.preventDefault();
    setActiveIndex(next);
    itemRefs.current[next]?.focus();
  };

  return (
    <div role="toolbar" aria-label="Text formatting">
      {actions.map((action, i) => (
        <button
          key={action.id}
          ref={(el) => { itemRefs.current[i] = el; }}
          tabIndex={i === activeIndex ? 0 : -1}
          onKeyDown={(e) => handleKeyDown(e, i)}
          aria-pressed={action.active}
        >
          {action.label}
        </button>
      ))}
    </div>
  );
}
```

---

### Combobox

| Key | Action |
| --- | ------ |
| Printable character | Filter list, open popup if closed |
| `Arrow Down` | If closed → open popup; if open → next option |
| `Arrow Up` | Previous option |
| `Enter` | Select highlighted option, close popup |
| `Escape` | Close popup without selecting; if already closed → clear field |
| `Tab` | Accept highlighted option OR move focus out (implementation choice) |

```tsx
function handleComboKeyDown(e: React.KeyboardEvent<HTMLInputElement>) {
  switch (e.key) {
    case "ArrowDown":
      e.preventDefault();
      setIsOpen(true);
      setActiveIndex((i) => Math.min(i + 1, filtered.length - 1));
      break;
    case "ArrowUp":
      e.preventDefault();
      setActiveIndex((i) => Math.max(i - 1, 0));
      break;
    case "Enter":
      e.preventDefault();
      if (isOpen && activeIndex >= 0) {
        selectOption(filtered[activeIndex]);
        setIsOpen(false);
      }
      break;
    case "Escape":
      if (isOpen) setIsOpen(false);
      else setValue("");
      break;
    case "Tab":
      if (isOpen && activeIndex >= 0) {
        selectOption(filtered[activeIndex]);
      }
      setIsOpen(false);
      break;
  }
}
```

---

### Data Grid

| Key | Action |
| --- | ------ |
| `Arrow Up/Down/Left/Right` | Move between cells |
| `Home` | First cell in current row |
| `End` | Last cell in current row |
| `Ctrl+Home` | First cell in first row |
| `Ctrl+End` | Last cell in last row |
| `Page Up` | Previous page (if paginated) |
| `Page Down` | Next page |
| `Enter` / `F2` | Enter edit mode (for editable grids) |
| `Escape` | Exit edit mode |
| `Space` | Select row (if row selection enabled) |

> Use roving tabindex within the `role="grid"` — only the active cell has `tabIndex={0}`.

---

## Roving tabindex

The pattern used throughout Tree View/Toolbar/Data Grid above: a composite
widget keeps **exactly one** tab stop for the whole widget, and arrow keys
move which item is "active" (`tabIndex={0}` on the active item, `-1` on the
rest, or a single `aria-activedescendant` pointer on the container instead of
moving real DOM focus). This is why Tab enters/exits the widget once instead
of stepping through every item — the widget manages internal navigation
itself via arrow keys, not the browser's native Tab order.

## Skip link

```html
<a href="#main" class="sr-only focus:not-sr-only focus:absolute focus:top-2 focus:left-2 focus:bg-bg focus:text-fg focus:px-3 focus:py-2 focus:rounded">Skip to main</a>
<main id="main" tabIndex={-1}>...</main>
```

The skip link is invisible until focused (Tab from the address bar), letting
keyboard users bypass nav. It should be the **first focusable** element in
the document (or immediately after a minimal banner). See
[`focus-management.md`](./focus-management.md) for the full `SkipLink`
component and the `tabIndex={-1}` requirement on the target.

## Character key shortcuts (2.1.4)

Single-letter shortcuts (e.g. `j`/`k` list navigation) must be:

- Turned off until the user enables them, **or**
- Active only when a specific component has focus, **or**
- Remappable.

Document shortcuts and provide a settings toggle when using global listeners on `document`.

## Advanced keyboard patterns

### Escape propagation hierarchy

Escape closes **the innermost dismissible layer** — one layer per press:

```ts
// Combobox dropdown inside a dialog
comboboxRef.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && dropdownOpen) {
    closeDropdown();
    e.stopPropagation(); // prevent dialog from closing on same Escape
  }
});
```

General rule: peel layers outward. Nested popup → parent popup → dialog → page overlay.

### F6 landmark region cycling

In application-style interfaces, F6 cycles between landmark regions (banner → navigation → main → complementary). The browser does not implement this natively — implement in JavaScript:

```ts
const LANDMARKS = ['banner', 'navigation', 'main', 'complementary', 'contentinfo', 'form', 'region'];
const landmarkEls = () => Array.from(document.querySelectorAll(
  'header[role], nav, main, aside, footer, [role="banner"], [role="navigation"], [role="main"], [role="complementary"]'
));

document.addEventListener('keydown', (e) => {
  if (e.key !== 'F6') return;
  const landmarks = landmarkEls();
  const current = landmarks.findIndex(el => el.contains(document.activeElement));
  const next = e.shiftKey
    ? (current - 1 + landmarks.length) % landmarks.length
    : (current + 1) % landmarks.length;
  (landmarks[next] as HTMLElement).focus?.() ?? landmarks[next].querySelector<HTMLElement>('h1,h2,h3,[tabindex="0"],button,a')?.focus();
});
```

Each landmark needs a unique `aria-label` when multiple of the same type exist (e.g. two `<nav>` elements).

### Printable character typeahead (listbox, tree, grid)

When a printable character is pressed in a listbox/tree, move focus to the next matching option:

```ts
let typeaheadBuffer = '';
let typeaheadTimer: number;

container.addEventListener('keydown', (e) => {
  if (e.key.length !== 1 || e.ctrlKey || e.metaKey) return; // not printable
  clearTimeout(typeaheadTimer);
  typeaheadBuffer += e.key.toLowerCase();
  typeaheadTimer = setTimeout(() => { typeaheadBuffer = ''; }, 300);

  const options = Array.from(container.querySelectorAll('[role="option"]'));
  const currentIdx = options.findIndex(o => o.id === activeId);
  // Search from current+1, wrapping
  const match = options.slice(currentIdx + 1).concat(options.slice(0, currentIdx + 1))
    .find(o => o.textContent?.toLowerCase().startsWith(typeaheadBuffer));
  if (match) setActive(match.id);
});
```

Multi-character buffer with 300ms reset allows typing 'SW' to jump to 'Sweden'.

### Multiselect listbox (Shift+Arrow)

```ts
// State: anchor index, set of selected ids
function handleKey(e: KeyboardEvent, currentIdx: number) {
  if (e.key === 'ArrowDown' && e.shiftKey) {
    const nextIdx = currentIdx + 1;
    extendSelection(anchorIdx, nextIdx); // select all from anchor to next
    moveFocus(nextIdx);
  }
  if (e.key === 'a' && e.ctrlKey) selectAll();
}
```

`aria-multiselectable="true"` on listbox; `aria-selected="true"/"false"` on **all** options (not just selected ones).

## Pitfalls

- **`onKeyDown` only handles Enter** — also handle Space for buttons-as-divs (or just use `<button>`).
- **Focus lost after content change** — when a list updates and the focused row disappears, focus jumps to `<body>`. Restore focus to a stable nearby element.
- **Dialog opens with focus on the first input that says "Email"** — gives a screen reader user no context. Focus the dialog title (or a wrapper with the dialog name) first — see [`focus-management.md`](./focus-management.md) for the auto-focus-override pattern.
- **Custom widget without keyboard handler** — the most common a11y failure. If you build it, you implement every keyboard interaction.

## Pointer Cancellation (WCAG 2.5.2 AA)

See [mobile-touch.md](./mobile-touch.md#pointer-cancellation-wcag-252-aa) for the full requirement, the `DraggableItem` example, and the `onClick` vs `onPointerDown` anti-pattern.

**Quick recap:** never fire the action on `pointerdown`/`mousedown` — always use `click` (fires on pointer-up, so the user can move off the target to cancel). For custom drag, cancel on `onPointerLeave`/`onPointerCancel`.
