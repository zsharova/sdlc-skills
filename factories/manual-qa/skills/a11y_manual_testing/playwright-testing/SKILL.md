---
name: playwright-manual-accessibility-testing
description: Run manual WCAG checks against a live web page through the Playwright MCP server, using criterion-specific reference files for test scope, evidence, and classification. Use when testing one or more WCAG criteria, or all bundled WCAG references, without generating Playwright test code. Execute criteria sequentially by default and pass stored results to a separate reporting step.
---

# Playwright Manual Accessibility Testing

Drive a real browser through Playwright MCP and execute WCAG checks defined in
criterion-specific files under `references/`. This skill performs evaluation
only. Reporting is a separate step.

## Inputs

Resolve:

- the exact URL to test;
- requested WCAG criteria, when specified;
- optional page scope, viewport, authentication state, or setup instructions;
- any explicit execution-order or stop condition.

Ask for a missing value only when it blocks execution. Do not invent a URL,
credential, test account, or WCAG criterion.

## Select WCAG references

Use criterion files named `references/<criterion>.md`, for example
`references/2.5.3.md`.

1. If the user specifies one or more criteria, normalize identifiers such as
   `SC 2.5.3` to `2.5.3`, preserve the requested order, and load only the
   matching reference files.
2. If a criterion file lists other required WCAG references, load those files
   before running that criterion.
3. If a requested reference is missing, mark that criterion `BLOCKED`; do not
   replace it with an improvised procedure.
4. If the user does not specify criteria, discover every numbered WCAG file in
   `references/` and order them by dotted criterion number using version-sort
   semantics.
5. Read each selected reference completely before executing it.

The active criterion reference defines its scope, testing mode, allowed
interaction, discovery method, evidence, and classification rules.

## Execute criteria sequentially

Run all selected criteria one after another unless the user explicitly requests
a different execution model.

For each criterion:

1. Load the criterion reference and any references it requires.
2. Open the exact target URL in Playwright.
3. Establish only the state allowed by that criterion reference.
4. Execute the complete criterion procedure.
5. Validate candidate processing and counts required by the reference.
6. Store the criterion result and evidence.
7. Close the page or browser as required by the reference.
8. Start the next criterion only after the current one finishes.

A WCAG failure is a finding, not an execution error: store it and continue with
the next criterion. If a technical condition blocks one criterion, record the
block and continue when later criteria can still run. Stop early only when the
user requests fail-fast behavior or the Playwright environment or target is
unavailable for all remaining criteria.

Do not reuse classifications or DOM observations from an earlier criterion.
Each criterion must satisfy its own discovery and coverage rules.

## Use Playwright MCP

- Drive the running page with Playwright MCP browser tools. Do not create or
  run `.spec.*`, pytest, Page Object Model, or other test-automation files.
- Navigate to the exact user-provided URL and store the actual tested URL.
- Use DOM inspection when the active reference requires DOM-based discovery.
- Use browser accessibility information when the active reference requires a
  computed accessible name or semantic role.
- Use the accessibility snapshot only within the limits defined by the active
  reference.
- Take a new browser snapshot before an allowed action and after navigation or
  a material DOM change; snapshot element references may become stale.
- Wait for observable application state instead of fixed delay when
  interaction is allowed.
- Never perform an interaction forbidden by the active criterion reference.

## Preserve evaluation integrity

- Apply only the active WCAG criterion.
- Give every in-scope candidate exactly one classification allowed by the
  reference.
- Keep `FAIL`, `NOT_APPLICABLE`, `NOT_EVALUATED`, `BLOCKED`, and technical
  execution errors distinct.
- Never convert missing or ambiguous evidence into `PASS`.
- Never override deterministic classification with AI judgment.
- Keep the stable locator and evidence fields required by the criterion.
- Mark a criterion complete only after its processing validation succeeds.

## Result hand-off

Store results for the separate reporting step in this structure:

```json
{
  "runStatus": "COMPLETE | PARTIAL | BLOCKED",
  "testedUrl": "exact URL analyzed by Playwright",
  "requestedCriteria": ["2.5.3"],
  "executionOrder": ["2.5.3"],
  "criteria": [
    {
      "criterion": "2.5.3",
      "title": "Label in Name",
      "level": "A",
      "executionStatus": "COMPLETE | INCOMPLETE | BLOCKED",
      "testingMode": "Pure Static",
      "counts": {},
      "results": [],
      "blockReason": null
    }
  ]
}
```

Use the criterion reference to populate `counts` and `results`. Set the overall
status to `PARTIAL` when at least one criterion completes and another is
incomplete or blocked. Set it to `BLOCKED` when none can run.

Do not:

- group findings into bugs;
- assign priority;
- generate reproduction steps or recommendations;
- render HTML or any other report;
- call a reporting skill;
- claim that no issues exist when processing is incomplete.

## Available criterion references

- [2.5.3 — Label in Name](references/2.5.3.md): initial rendered state,
  DOM-based candidate discovery, visible-label detection, browser-computed
  accessible names, and deterministic comparison.
