---
name: Project briefing
description: Stack overlay (feature-development/web) — React frontend defaults, accessibility on request; scout refines per project
type: project
---

## Project Knowledge

- **Stack:** React frontend (TS), talking to the backend over the API
  contract `tech-lead` owns. _(confirm exact framework/build tool from
  AGENTS.md)_
- **Accessibility is a non-functional requirement, invoked on request — not a
  default applied to every task.** When the ticket, `tech-lead`, or the user
  explicitly calls for WCAG/a11y compliance (or asks to "make this
  accessible"), use the `a11y-dev` skill for technique (including how to
  build accessible tabs/comboboxes/dialogs/menus). Don't proactively apply
  a11y work to tasks that didn't ask for it.

## My Role Focus

Own the frontend: UI, client state, consuming the API contract `tech-lead`
coordinates. Follow existing component patterns before introducing new ones.
When a task explicitly calls for accessibility:
- Reach for the project's accessible-primitives library (Radix/shadcn,
  Headless UI, etc.) if one exists, rather than hand-rolling ARIA/keyboard
  behavior from scratch.
- Spot-check with `browser-verify`'s `inject-axe` before calling the work
  done — it catches missing labels/roles and contrast issues, but not
  focus-order or keyboard-trap problems, which need a manual Tab-through.
- Expect `tech-lead`'s review to include the Accessibility category in
  `code-review`, and possibly a full `a11y-audit` pass, when accessibility
  was part of the requirement.
