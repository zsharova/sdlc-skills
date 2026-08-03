# Angular A11y Patterns

## CDK tools — always prefer over manual ARIA management

```typescript
// LiveAnnouncer — for dynamic content that must be announced
import { LiveAnnouncer } from '@angular/cdk/a11y';

@Component({...})
export class MyComponent {
  constructor(private announcer: LiveAnnouncer) {}

  onSave() {
    this.saveData();
    this.announcer.announce('Changes saved successfully', 'polite');
    // Use 'assertive' only for errors or urgent interruptions
  }
}
```

```typescript
// cdkTrapFocus — for modals, drawers, and any overlay
// template:
// <div cdkTrapFocus cdkTrapFocusAutoCapture>
//   ... modal content ...
// </div>
```

```typescript
// FocusMonitor — detect keyboard vs mouse focus (focus-visible equivalent)
import { FocusMonitor } from '@angular/cdk/a11y';

@Component({...})
export class MyButton implements OnDestroy {
  constructor(private fm: FocusMonitor, private el: ElementRef) {
    this.fm.monitor(this.el).subscribe(origin => {
      this.isKeyboardFocused = origin === 'keyboard';
    });
  }
  ngOnDestroy() { this.fm.stopMonitoring(this.el); }
}
```

## ARIA attribute binding

```html
<!-- CORRECT: use attr. prefix for non-boolean ARIA -->
<div [attr.aria-expanded]="isOpen" [attr.aria-label]="label">

<!-- CORRECT: boolean ARIA -->
<button [attr.aria-pressed]="isPressed">Toggle</button>

<!-- WRONG: Angular warns — do not use -->
<div [aria-expanded]="isOpen">
```

## Router accessibility — announce route changes

```typescript
// In AppComponent
import { Router, NavigationEnd } from '@angular/router';
import { LiveAnnouncer } from '@angular/cdk/a11y';

constructor(private router: Router, private announcer: LiveAnnouncer) {
  this.router.events
    .pipe(filter(e => e instanceof NavigationEnd))
    .subscribe(() => {
      // Give the new page title time to render
      setTimeout(() => {
        const title = document.title;
        this.announcer.announce(`Navigated to ${title}`, 'polite');
      }, 100);
    });
}
```

## Active link — aria-current

```html
<a routerLink="/dashboard"
   routerLinkActive="active"
   [ariaCurrentWhenActive]="'page'">
  Dashboard
</a>
```

## Angular Material form fields

```html
<!-- CORRECT — mat-label provides accessible name -->
<mat-form-field>
  <mat-label>Email address</mat-label>
  <input matInput type="email" autocomplete="email" />
  <mat-error>Please enter a valid email</mat-error>
</mat-form-field>

<!-- If no mat-label, must use aria-label on input -->
<mat-form-field>
  <input matInput aria-label="Search" placeholder="Search..." />
</mat-form-field>
```

## Roving tabindex / arrow-key navigation — CDK Key Managers

For custom composite widgets (menus, comboboxes, trees, toolbars), use `@angular/cdk/a11y`'s
key managers instead of hand-rolling arrow-key handlers. This is the Angular equivalent of the
React roving-tabindex pattern in [keyboard-navigation.md](./keyboard-navigation.md#roving-tabindex).

```typescript
import { FocusKeyManager } from '@angular/cdk/a11y';
import { QueryList, ViewChildren, AfterViewInit } from '@angular/core';

@Component({...})
export class MenuComponent implements AfterViewInit {
  @ViewChildren(MenuItemComponent) items!: QueryList<MenuItemComponent>;
  private keyManager!: FocusKeyManager<MenuItemComponent>;

  ngAfterViewInit() {
    // withWrap() makes Arrow keys cycle past the first/last item
    this.keyManager = new FocusKeyManager(this.items).withWrap();
  }

  onKeydown(event: KeyboardEvent) {
    this.keyManager.onKeydown(event); // handles ArrowUp/Down, Home, End, typeahead
  }
}
```

`ActiveDescendantKeyManager` is the equivalent for `aria-activedescendant`-based widgets
(listbox/combobox) where items should never receive real DOM focus — same distinction as the
`tabIndex` vs `aria-activedescendant` choice documented in aria-patterns.md's virtual-list section.

## ESLint — install for Angular

```bash
# Angular templates are HTML, not JSX — eslint-plugin-jsx-a11y does not apply here.
# Use angular-eslint's template plugin, which ships its own a11y rule set:
npm install --save-dev @angular-eslint/eslint-plugin-template
```

`.eslintrc.json`:
```json
{
  "plugins": ["@angular-eslint/template"],
  "rules": {
    "@angular-eslint/template/alt-text": "error",
    "@angular-eslint/template/label-has-associated-control": "error",
    "@angular-eslint/template/interactive-supports-focus": "error",
    "@angular-eslint/template/role-has-required-aria-props": "error"
  }
}
```
