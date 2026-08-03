---
name: code-review
description: Review code for correctness, security, performance, maintainability, and accessibility. Use when the user asks to "review this code", "check my changes", "review PR", "audit" — and proactively after a non-trivial change or before opening, approving, or merging a PR, even unprompted.
license: Apache-2.0
compatibility: Requires git CLI. Optional gh CLI for PR reviews.
metadata:
  authors:
    - Artem Rozumenko <artem_rozumenko@epam.com>
  version: "0.1.0"
---

# Code Review

Systematic code review focused on what matters. Not a style police — a correctness and risk checker.

## Review Scope

Determine what to review:

```bash
# Review staged changes
git --no-pager diff --cached

# Review working tree changes
git --no-pager diff

# Review a PR
gh pr diff <number>

# Review specific files
# Just read them directly
```

## Review Checklist

Check in this priority order. Stop at each category — don't rush to the next.
Categories 1-5 always apply; category 6 (Accessibility) only when explicitly
in scope (see below).

### 1. Correctness
- Does it do what it claims?
- Are edge cases handled (nulls, empty collections, boundary values)?
- Are error paths correct (not swallowed, not over-caught)?
- Do conditional branches cover all cases?

### 2. Security
- Input validation at system boundaries (user input, API responses)?
- SQL injection, XSS, command injection risks?
- Secrets in code or logs?
- Permissions and authorization checked?

### 3. Concurrency & State
- Race conditions in shared state?
- Proper locking/synchronization?
- Async operations awaited correctly?
- Resources properly closed/released?

### 4. Performance
- O(n^2) or worse in hot paths?
- Unnecessary allocations in loops?
- N+1 query patterns?
- Missing indexes for new queries?

### 5. Maintainability
- Is the intent clear from reading the code?
- Could someone unfamiliar modify this safely?
- Are there implicit assumptions that should be explicit?

### 6. Accessibility (only when explicitly in scope)
Accessibility is a non-functional requirement, not a default check applied to
every review — run this category only when the ticket/NFR/user explicitly
calls for WCAG/a11y compliance or an accessibility review. Skip it otherwise;
don't treat it as an implicit blanket requirement on UI changes that never
asked for it.

When it is in scope:
- Semantic HTML used over ARIA-patched `<div>`s where a native element exists?
- Every new interactive element keyboard-operable, with a visible focus
  indicator and an accessible name (icon-only buttons are the most common miss)?
- Any regression to keyboard/focus behavior (removed focus trap, lost return-focus on modal close, `outline: none` with no replacement)?
- Form inputs labeled, errors associated and not color-only?
- Were a11y unit tests (`vitest-axe` / `jest-axe` / Angular `axe-core` against
  a `TestBed` fixture) added for any new interactive component in this diff?

For technique guidance on any of the above, see `../a11y-dev/SKILL.md`. This
category is a quick pass, not a full audit — for a full gated WCAG review,
component-library risk check, and report, run the sibling skill
`../a11y-audit/SKILL.md`.

## Output Format

Structure your review as:

```
## Summary
One sentence: what this change does and whether it looks good.

## Issues
### [Critical/Important/Nit] Title
File: path/to/file.py:42
Description of the issue and why it matters.
Suggested fix (if non-obvious).

## Questions
Things that aren't wrong but need clarification from the author.
```

Severity levels:
- **Critical** — will break in production, security vulnerability, data loss risk
- **Important** — correctness issue, significant performance problem, missing error handling
- **Nit** — style, naming, minor improvement (limit to 2-3 max, don't pile on)

## What NOT to Review

- Style preferences already handled by formatters/linters
- Adding type hints to code the author didn't change
- Suggesting refactors unrelated to the change
- Commenting on test structure unless tests are wrong
