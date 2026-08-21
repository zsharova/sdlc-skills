# Shared Dynamic State Testing

## Contents

- [Purpose](#purpose)
- [Contract](#contract)
- [Scope](#scope)
- [Establish the clean baseline](#1-establish-the-clean-baseline)
- [Discover initial triggers](#2-discover-initial-triggers)
- [Filter eligible triggers](#3-filter-eligible-triggers)
- [Store the trigger inventory](#4-store-the-trigger-inventory)
- [Process triggers independently](#5-process-triggers-independently)
- [Compare before and after](#6-compare-before-and-after)
- [Expose the dynamic state](#7-expose-the-dynamic-state)
- [Reset to baseline](#8-reset-to-baseline)
- [Validate dynamic processing](#9-validate-dynamic-processing)
- [Return dynamic coverage](#10-return-dynamic-coverage)

## Purpose

Establish independent dynamic rendered states for an active WCAG criterion.
Discover and activate reveal-only triggers, compare the page before and after
activation, expose confirmed changed states to the criterion algorithm, and
restore the clean baseline between triggers.

Do not determine WCAG applicability, `PASS`, `FAIL`, `NOT_APPLICABLE`, or
`NOT_EVALUATED` in this reference.

## Contract

Receive from the parent skill:

- an open Playwright page at the criterion baseline;
- the exact `testedUrl`;
- the active criterion identifier;
- `interactionProfile = reveal-only`;
- `stateScope = initial-triggers-only`;
- `resetStrategy = reload-baseline`;
- optional user-provided setup instructions that do not perform the tested
  interaction.

Return confirmed dynamic states one at a time to the active criterion
algorithm. Return processing evidence and coverage counts to the parent skill.
Do not require dynamic-state instructions inside the criterion reference.

Require the parent to load `patterns.md` before this procedure. Apply its
`reveal-only` interaction profile, snapshot, target, wait, recovery, and
return contracts without restating them here.

## Scope

Use only triggers discovered in the clean initial rendered state. Do not
discover additional triggers inside an opened dialog, menu, tab panel,
accordion panel, popover, or other dynamic state.

Activate one initial trigger at a time. Do not combine triggers or traverse a
state tree.

Apply only the `reveal-only` interaction profile defined in `patterns.md`.

## 1. Establish the clean baseline

Use the initial rendered page state established for the active criterion.
Establish page readiness under the loaded `patterns.md` contract.

Store a baseline signature sufficient to verify reset. Include:

- actual URL;
- viewport;
- visible dialog, menu, tab panel, accordion, popover, and disclosure state;
- relevant DOM structure or visible-region signature;
- user-provided non-test setup state, when applicable.

Build a baseline comparison index from the active criterion's candidate
definition. For each criterion candidate retain:

- stable DOM identity or stable locator;
- baseline existence;
- baseline visibility;
- every available baseline evidence value required by the active criterion for
  comparison and classification.

Obtain the baseline snapshot under the loaded `patterns.md` contract before
trigger discovery.

## 2. Discover initial triggers

Inspect the baseline DOM for visible, enabled elements whose semantics or
attributes indicate that activation may reveal, hide, replace, or update
content in the current page.

Include potential triggers such as:

```css
button,
input[type="button"],
[role="button"],
[role="tab"],
[role="combobox"],
[aria-expanded],
[aria-controls],
[aria-haspopup],
summary,
[popovertarget],
[data-toggle],
[data-bs-toggle]
```

Use DOM discovery as the source of the trigger inventory. Use Playwright
snapshots and accessibility information as supplementary evidence.

## 3. Filter eligible triggers

Retain only visible and enabled triggers that can be exercised within the
`reveal-only` interaction profile.

Apply all `reveal-only` exclusions and safety boundaries from `patterns.md`.
Also exclude triggers explicitly excluded by the user or criterion reference.

If a potential trigger cannot be classified as reveal-only from available
evidence, retain it as `not_evaluated_trigger`. Do not activate it.

## 4. Store the trigger inventory

Store the eligible initial triggers before activating any of them.

For each potential trigger retain:

- stable locator;
- rendered text or visible label;
- computed accessible name, when available;
- semantic role;
- relevant reveal attributes;
- eligibility: `eligible | excluded | not_evaluated_trigger`;
- exclusion or uncertainty reason, when applicable;
- discovery order.

Do not add triggers discovered after activation to this inventory.

## 5. Process triggers independently

Process eligible triggers in stored discovery order.

For each trigger:

1. Restore and verify the clean baseline.
2. Resolve the stored stable locator against the refreshed DOM.
3. Take fresh before-state DOM and Playwright evidence.
4. Activate the trigger once using the primary action allowed by the loaded
   `patterns.md` contract.
5. Wait for the observable state change under that contract.
6. Obtain fresh after-state DOM and Playwright evidence under that contract.
7. Compare the before and after states.
8. Expose a confirmed changed state to the active criterion evaluation steps.
9. Store the complete returned criterion state result.
10. Compare the returned candidate evidence with the baseline comparison
    index and retain the dynamic delta.
11. Restore and verify the clean baseline before processing the next trigger.

Do not activate another trigger while the current trigger state is open.

## 6. Compare before and after

Determine whether activation produced a material rendered-state change.

Retain evidence for:

- DOM nodes added or removed;
- elements becoming visible or hidden;
- visible text changes;
- accessible name or semantic changes available from browser information;
- dialog, menu, tab, accordion, popover, disclosure, or expanded-state
  changes;
- trigger changes relevant to the active criterion.

For every active-criterion candidate in a confirmed changed state, assign one
comparison category:

- `new`: the candidate did not exist in the baseline comparison index;
- `newly_visible`: the candidate existed but was not visible in the baseline;
- `changed`: the candidate was visible and at least one criterion-required
  evidence value changed;
- `unchanged`: the candidate and its criterion-required evidence match the
  baseline.

Perform this comparison without changing the criterion result. Removed or
newly hidden candidates remain state-change evidence but do not become visible
dynamic candidates unless the active criterion says otherwise.

Assign one state outcome:

- `changed`: a material rendered-state change is confirmed;
- `no_change`: activation completed but no material change is confirmed;
- `activation_failed`: the trigger could not be activated;
- `state_not_evaluated`: technical evidence is insufficient to determine the
  state outcome.

Do not treat `no_change`, `activation_failed`, or `state_not_evaluated` as a
WCAG result.

## 7. Expose the dynamic state

For every `changed` outcome, create and pass this state context to the active
criterion algorithm:

```json
{
  "stateId": "criterion-trigger-sequence",
  "stateType": "dynamic",
  "trigger": {
    "order": 1,
    "locator": "stable locator",
    "visibleLabel": "rendered trigger label",
    "accessibleName": "computed name when available",
    "role": "semantic role",
    "action": "activate"
  },
  "change": {
    "outcome": "changed",
    "regions": [],
    "beforeEvidence": {},
    "afterEvidence": {}
  }
}
```

Keep the state open until the active criterion evaluation steps return their
complete stored result. Do not classify WCAG candidates or modify any returned
criterion classification.

After candidate comparison, retain results for `new`, `newly_visible`, and
`changed` candidates as the dynamic delta. Keep `unchanged` candidates only for
state-processing validation. Do not aggregate unchanged baseline candidates a
second time.

## 8. Reset to baseline

After storing the current state result, reload the exact `testedUrl`. Reapply
only user-provided non-test setup instructions required to reproduce the
baseline.

Re-establish readiness under `patterns.md`. Verify the baseline signature
before processing the next stored trigger.

If reload or baseline verification fails:

- store `reset_failed` for the current trigger;
- stop dynamic trigger processing for the active criterion;
- return the completed state results and incomplete dynamic coverage to the
  parent skill.

Do not continue from an unverified state.

## 9. Validate dynamic processing

Retain counts for:

- potential initial triggers;
- eligible initial triggers;
- excluded initial triggers;
- not-evaluated initial triggers;
- attempted triggers;
- changed states;
- no-change triggers;
- activation failures;
- state-not-evaluated triggers;
- reset failures;
- criterion state results returned.

Verify:

- every potential initial trigger has one eligibility outcome;
- every eligible trigger has one processing outcome;
- every changed state has one returned criterion state result;
- every processed trigger starts from a verified baseline;
- no trigger discovered after baseline is processed;
- no state contains more than one activated initial trigger.

Mark dynamic processing incomplete when any required trigger or reset cannot be
processed consistently. Do not produce a complete dynamic-coverage conclusion
from inconsistent processing.

## 10. Return dynamic coverage

Return to the parent skill:

- the stored initial trigger inventory;
- all trigger processing outcomes;
- all state contexts;
- all returned criterion state results;
- all retained dynamic deltas and candidate comparison categories;
- dynamic coverage counts;
- dynamic processing status: `COMPLETE | INCOMPLETE | BLOCKED`;
- block or incomplete reason, when applicable.

Do not group findings, assign priority, generate bug content, create
recommendations, or render a report.
