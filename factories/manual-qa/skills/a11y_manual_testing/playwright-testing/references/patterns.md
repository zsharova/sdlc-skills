# Playwright MCP Interaction Patterns

## Contents

- [Purpose](#purpose)
- [Contract precedence](#contract-precedence)
- [Required inputs](#required-inputs)
- [Page establishment and blocking](#page-establishment-and-blocking)
- [Browser lifecycle](#browser-lifecycle)
- [Snapshot and target contract](#snapshot-and-target-contract)
- [Wait and state contract](#wait-and-state-contract)
- [Interaction profiles](#interaction-profiles)
- [Interaction recipes](#interaction-recipes)
- [Framework-specific selector hints](#framework-specific-selector-hints)
- [Hybrid API and UI checks](#hybrid-api-and-ui-checks)
- [Screenshot convention](#screenshot-convention)
- [Diagnostics](#diagnostics)
- [Common gotchas](#common-gotchas)
- [Return contract](#return-contract)

## Purpose

Drive a running web application through Playwright MCP without generating test
code. Reuse this contract for static setup, dynamic state activation, keyboard
checks, form interaction, pointer interaction, and other criterion-specific
procedures.

Keep browser execution separate from WCAG evaluation. Do not determine
`PASS`, `FAIL`, `NOT_APPLICABLE`, or `NOT_EVALUATED` in this reference.

## Contract precedence

Apply instructions in this order:

1. explicit user restrictions;
2. active WCAG criterion reference;
3. registered interaction profile;
4. this shared Playwright contract;
5. an applicable recipe below.

A recipe describes how to perform an action when that action is permitted. It
does not authorize the action.

Never use a broader recipe to override a criterion's no-click, no-type,
no-hover, no-focus, no-navigation, or no-state-change restriction.

Use Playwright MCP browser tools against the running page. Do not create or run
`.spec.*`, pytest, Page Object Model, or other test-automation files.

## Required inputs

Receive from the parent skill or active criterion:

- the exact target URL;
- the active criterion identifier;
- the registered interaction profile;
- the current project root;
- the run timestamp and output directory when evidence files are required;
- optional page scope, viewport, authentication state, and setup instructions;
- explicit stop, reset, cleanup, or fail-fast conditions.

Do not invent a URL, credential, test account, personal data, file, expected
application content, or allowed interaction.

Store the actual URL reached by Playwright. Keep credentials, cookies, tokens,
authorization headers, and session data out of stored evidence and reports.

## Page establishment and blocking

Make the parent skill responsible for establishing the page. Criterion
references receive an established rendered state and must not open the target
URL themselves.

To establish a page:

1. open the exact user-provided URL;
2. store the actual URL reached as `testedUrl`;
3. apply only supplied viewport, authentication, and setup instructions;
4. wait for observable page readiness;
5. take the initial snapshot and required DOM or accessibility evidence;
6. pass the established rendered state and `testedUrl` to the active
   criterion.

If authentication, security protection, network failure, application load
failure, or another condition prevents the page from being established:

- do not invent or bypass credentials, security controls, or setup;
- do not interact with blocking UI unless supplied setup instructions and the
  active interaction profile explicitly permit it;
- store `browserExecutionStatus = BLOCKED`;
- store the exact blocking reason and available evidence;
- close the page and browser under the lifecycle contract;
- return the blocked state to the parent.

Do not call a criterion evaluator when its required rendered state was not
established.

## Browser lifecycle

Make the parent skill the lifecycle owner for the Playwright page, browser, and
session. Criterion references must not open, reload for orchestration, or close
Playwright.

Keep the required page open until every evaluator and shared state procedure
that needs the current browser state has returned its stored evidence.

For a static run:

1. establish the page;
2. run the criterion evaluator;
3. store and validate its evidence;
4. close the page and browser.

For a dynamic run:

1. establish the baseline page;
2. run the baseline evaluator;
3. execute all required independent state procedures and resets;
4. store and validate every returned state result;
5. close the page and browser after dynamic coverage ends.

On `COMPLETE`, `INCOMPLETE`, or `BLOCKED` execution:

- store all available evidence and counts before closing;
- perform only required safe cleanup;
- close the Playwright page;
- close the Playwright browser or session;
- store the close outcome.

After closure, use stored evidence only. Do not reopen or inspect the tested
page for classification, grouping, reporting preparation, or report rendering.

## Snapshot and target contract

Use `browser_snapshot` before resolving an actionable element.

For every browser action:

- use the target from the latest browser snapshot;
- pass a human-readable `element` description with the `target`;
- use a snapshot ref when available;
- use a unique CSS selector when a snapshot ref is unavailable or ambiguous;
- retain a stable locator when the element must be resolved again after reload
  or reset.

Treat snapshot refs as invalid after:

- navigation;
- reload;
- baseline reset;
- opening or closing a dialog, menu, popover, listbox, or disclosure;
- a material DOM or accessibility-tree change.

After any such event:

1. take a new snapshot;
2. resolve the element again;
3. use the fresh target;
4. refresh DOM and accessibility evidence required by the active criterion.

Use role and accessible name for locating when they are available and unique.
Use framework-specific selectors only as fallback locators. Do not use locator
convenience as WCAG evidence when the active criterion requires DOM discovery
or browser-computed accessibility information.

## Wait and state contract

Wait for an observable application condition. Do not use a fixed delay as the
primary readiness or success condition.

After an allowed action, wait for one or more of:

- expected text appearing;
- transient text disappearing;
- a visibility change;
- a DOM addition, removal, or attribute change;
- an accessibility-tree or accessible-name change;
- a dialog, menu, listbox, tab panel, accordion, popover, or disclosure state
  change;
- a URL change when navigation is explicitly permitted.

Take a fresh snapshot after the observable condition is reached.

If no observable condition can be defined, store the limitation. Do not infer
success from elapsed time or a successful tool response alone.

Keep technical action outcomes separate from WCAG results.

## Interaction profiles

Use exactly the interaction profile registered by the parent skill. Do not mix
profiles unless the active criterion explicitly requires the combination.

### `none`

Perform no click, typing, hover, focus movement, submission, selection, upload,
navigation, or state change after the initial page is established.

### `reveal-only`

Allow activation only when it reveals, hides, replaces, or updates content
without navigation, submission, persistent data mutation, download,
authentication change, or external application launch.

Allow opening an in-page dialog, menu, tab panel, accordion, popover,
disclosure, listbox, or custom combobox.

For a custom dropdown or combobox:

- open the component;
- inspect the revealed options or content;
- do not select an option unless another profile explicitly permits selection.

Exclude or do not evaluate a trigger when available evidence cannot confirm
that the action is reveal-only.

### `keyboard`

Allow keyboard navigation and keys explicitly required by the active criterion.
Take a fresh snapshot after focus movement or state change. Do not type content
or submit unless the profile also permits it.

### `focus-hover`

Allow focus or hover only when required by the active criterion. Restore the
baseline focus or pointer state after each independent check.

### `form-input`

Allow typing, native selection, custom option selection, and submission only
when the active criterion defines the input data, expected state, cleanup, and
data-mutation boundary.

### `pointer`

Allow pointer activation, movement, dragging, or target-size interaction only
as defined by the active criterion. Do not infer keyboard behavior from pointer
results.

### `timed-content`

Allow observation or control of time-dependent content only as defined by the
active criterion. Prefer observable state changes to fixed delays.

### `file-upload`

Allow upload only when the user or active criterion supplies the exact safe
file and cleanup instructions. Never invent or upload personal or sensitive
content.

### `data-setup`

Allow API, seed, or other out-of-band setup only when explicitly authorized.
Keep setup and teardown distinct from browser-visible evaluation.

## Interaction recipes

Each recipe is an explicit Playwright MCP tool chain. Replace placeholders with
supplied values. Use the recipe only when the registered profile permits every
action in the chain.

### Verify a page loads

```text
browser_navigate(url:"{{base_url}}/path")
browser_snapshot()
browser_take_screenshot(
  type:"png",
  filename:"reports/{runTimestamp}/screenshots/{TC_ID}_{date}.png"
)
```

Confirm required page readiness from observable content or structure.

### Log in once

Use only supplied credentials or an already authenticated browser state.

```text
browser_navigate(url:"{{base_url}}/login")
browser_snapshot()
browser_fill_form(fields=[
  {element:"Email field",    target:"<fresh ref>", name:"Email",    type:"textbox", value:"<supplied email>"},
  {element:"Password field", target:"<fresh ref>", name:"Password", type:"textbox", value:"<supplied password>"}
])
browser_click(element:"Sign in button", target:"<fresh ref>")
browser_wait_for(text:"<supplied authenticated-state text>")
browser_snapshot()
```

Do not re-login between steps of the same case while the session remains valid.
Reapply login only when baseline restoration requires it. Do not store supplied
credentials in evidence.

### Submit a form

Use only under `form-input` with defined test data and cleanup.

```text
browser_snapshot()
browser_fill_form(fields=[
  …one entry per field using fresh targets…
])
browser_click(element:"Submit button", target:"<fresh ref>")
browser_wait_for(text:"<supplied success text>")
browser_snapshot()
browser_take_screenshot(
  type:"png",
  filename:"reports/{runTimestamp}/screenshots/{TC_ID}_{date}.png"
)
```

Use `browser_wait_for(textGone:"Saving…")` when disappearance of transient
content is the observable completion condition.

For a single field, use:

```text
browser_type(
  element:"<field description>",
  target:"<fresh ref>",
  text:"<supplied value>",
  submit:true
)
```

Treat `submit:true` as submission. Do not use it outside a profile that permits
submission.

### Native dropdown

Use `browser_select_option` only for a native `<select>`.

```text
browser_snapshot()
browser_select_option(
  element:"Priority select",
  target:"<fresh ref>",
  values:["High"]
)
browser_snapshot()
```

Require `form-input` or another profile that explicitly permits selection.

### Custom dropdown or combobox

Do not use `browser_select_option` for a custom component.

To open the component:

```text
browser_snapshot()
browser_click(
  element:"Priority combobox",
  target:"<fresh trigger ref>"
)
browser_wait_for(text:"<supplied option or opened-state text>")
browser_snapshot()
```

Under `reveal-only`, stop after the opened-state snapshot.

To select an option when selection is permitted:

```text
browser_click(
  element:"Option: High",
  target:"<fresh option ref>"
)
browser_snapshot()
```

Resolve the option from the snapshot taken after the combobox opens.

### Drive an interactive element

```text
browser_snapshot()
browser_click(element:"<element description>", target:"<fresh ref>")
browser_wait_for(text:"<expected change>")
browser_snapshot()
browser_console_messages(level:"error")
```

Replace the wait condition with another observable condition when text is not
appropriate.

Treat console errors as technical diagnostic evidence only. Do not convert them
into WCAG findings.

### Upload a file

Use only under `file-upload`.

```text
browser_snapshot()
browser_click(element:"Choose file", target:"<fresh ref>")
browser_file_upload(paths:["/absolute/supplied/path/to/file.png"])
browser_wait_for(text:"file.png")
browser_snapshot()
```

Confirm that the selected file is safe and explicitly supplied. Apply required
cleanup after evaluation.

### Handle a native JavaScript dialog

When an alert, confirm, or prompt is expected and permitted:

```text
browser_handle_dialog(accept:true)
```

Use `accept:false` to dismiss. Supply `promptText:"…"` only when explicit
input is provided.

When a native dialog appears unexpectedly during a profile that does not
permit it:

1. dismiss it when safe;
2. store the trigger and technical outcome;
3. do not treat the dialog as a successful tested state;
4. restore and verify the baseline;
5. return control to the active state procedure.

Do not accept an unexpected confirmation that may mutate data.

## Framework-specific selector hints

Use these only when role and accessible name are ambiguous and a unique
snapshot ref is unavailable. A unique selector is a valid `target`.

### MUI

- Select without `label-for`: use `.MuiSelect-root` inside the field wrapping
  the label; locate options with `li[role="option"]` by text.
- Chip: use `.MuiChip-root` carrying the label.
- Dialog: use `[role="dialog"]`.
- Autocomplete: use `.MuiAutocomplete-root input`; locate results with
  `.MuiAutocomplete-option` by text.

### shadcn/ui

- Command or combobox: use `[cmdk-input]`; locate items with `[cmdk-item]`
  by text.
- Select: use `button[role="combobox"]`; locate options with
  `[role="option"]`.
- Dialog: use `[role="dialog"]`.
- Toast: use `[data-sonner-toast]`.

### Ant Design

- Select: use `.ant-select-selector`; locate options with
  `.ant-select-item-option`.
- Modal: use `.ant-modal-content`.
- Table row: use `.ant-table-row`.

For custom selects and comboboxes, use the custom dropdown recipe. Do not use
`browser_select_option` unless the target is a native `<select>`.

## Hybrid API and UI checks

Use only under `data-setup`.

```text
(set up data through the supplied application API or seeded account)
browser_navigate(url:"{{base_url}}/items/{id}")
browser_wait_for(text:"<supplied item name>")
browser_snapshot()
browser_take_screenshot(
  type:"png",
  filename:"reports/{runTimestamp}/screenshots/{TC_ID}_{date}.png"
)
(tear down the data through the same authorized channel)
```

Keep setup and teardown outside the browser. Use the browser for the
user-visible evaluation. Do not include secrets or raw API responses in the
accessibility evidence.

## Screenshot convention

Take screenshots only when the parent or active criterion requires them.

Name screenshots by run, criterion or case, and state:

```text
reports/{runTimestamp}/screenshots/{TC_ID}_{YYYY-MM-DD}.png
reports/{runTimestamp}/screenshots/login-initial.png
reports/{runTimestamp}/screenshots/login-error-invalid-password.png
reports/{runTimestamp}/screenshots/{criterion}_{stateId}.png
```

Treat filenames as relative to the Playwright MCP output directory. Keep
screenshots for one run inside that run's timestamp directory.

Do not use a screenshot as the sole evidence when the active criterion requires
DOM or browser accessibility information.

## Diagnostics

Use `browser_console_messages(level:"error")` or
`browser_network_requests` only when an allowed action appears to fail or the
active procedure requests technical diagnostics.

Store console and network evidence as technical execution data. Do not:

- create WCAG findings from unrelated console or network errors;
- change a deterministic WCAG classification;
- expose credentials, tokens, request bodies, cookies, or authorization data;
- treat the absence of errors as proof of WCAG conformance.

## Common gotchas

| Problem | Cause | Required response |
|---|---|---|
| Element not found | Action uses a stale ref | Take a new snapshot and use the fresh target |
| `target` rejected | Ref came from an earlier snapshot | Snapshot again after every navigation or material DOM change |
| Step succeeds but nothing changes | Success was inferred from the tool response or elapsed time | Wait for an observable application condition |
| Authentication is lost | Session expired or navigation changed the state | Reapply the supplied login or setup recipe |
| In-page modal blocks interaction | Overlay intercepts the action | Act within the current `[role="dialog"]` only when the profile permits it |
| Native dialog blocks the run | Alert, confirm, or prompt is unhandled | Handle it according to the native-dialog recovery contract |
| Native selection does nothing | Target is a custom combobox | Use the custom dropdown recipe |
| UI appears correct but the step fails | A silent application or network error occurred | Inspect console and network diagnostics without changing WCAG classification |
| Reset resolves the wrong element | Locator was not stable across reload | Resolve the stored stable locator against a fresh snapshot |
| Screenshot and DOM disagree | Screenshot or DOM evidence is stale | Refresh both and use evidence from the same rendered state |
| Action would exceed the profile | Recipe is broader than the active criterion | Do not perform the action; store the restriction or not-evaluated outcome |

## Return contract

Return to the parent procedure:

- actual tested URL;
- applied interaction profile;
- actions attempted in execution order;
- human-readable element descriptions and stable locators;
- fresh target evidence used for each action;
- observable wait conditions and outcomes;
- before and after snapshots or stored evidence when required;
- technical failures and recovery outcomes;
- screenshot paths when created;
- final browser and session state;
- cleanup or reset status.

Do not classify WCAG results, group findings, prepare bugs, or generate a
report. Return control to the active criterion or shared state procedure.
