---
name: accessibility-report
description: >
  Generates a standalone HTML accessibility report from confirmed
  accessibility testing results supplied by another accessibility skill.
  This skill is presentation-only and must not perform accessibility
  testing, determine PASS/FAIL, or modify supplied findings.
---

# Accessibility Report

## Purpose

Generate a standalone HTML accessibility report from confirmed results
supplied by a parent accessibility testing skill.

This skill is responsible only for:

- report presentation;
- HTML generation;
- visual layout;
- formatting supplied bug data.

It MUST NOT:

- perform accessibility testing;
- open or inspect the tested page;
- determine PASS or FAIL;
- change supplied classifications;
- change supplied evidence;
- invent findings;
- add WCAG issues;
- remove confirmed findings.

Use only the data supplied by the parent skill.

---

# Input Contract

The parent skill supplies:

## Metadata

- WCAG criterion
- WCAG level
- Tested URL
- Execution date/time
- Testing mode
- Analysis method
- Output path

## Summary

- Elements evaluated
- Passed
- Failed
- Bugs generated

## Bugs

For every grouped bug:

- Title
- WCAG
- Priority
- Steps to Reproduce
- Actual Result
- Expected Result
- Affected Elements
- Recommendations
- Techniques

## Affected Elements

Each affected element may contain:

- Visible Label
- Role
- Locator
- Accessible Name

Do not recalculate or reinterpret any supplied accessibility result.

---

# Output Location

The HTML report MUST be written inside the current Claude Code project.

The parent skill supplies a project-relative output path.

Example:

`./reports/wcag-2.5.3-label-in-name.html`

Interpret this path relative to the current project root.

The current project root is the directory from which the active
Claude Code project/session is being run.

Do NOT resolve the output path relative to:

- this SKILL.md file;
- `.claude/`;
- `.claude/skills/`;
- the accessibility-report skill directory;
- the user's global `.claude` directory;
- a temporary directory.

Before writing the report:

1. determine the current project root;
2. resolve the supplied output path from the project root;
3. create the `reports` directory if it does not exist;
4. write the HTML file to the resolved location.

For example, if the current project root is:

`C:\Users\Mariana_Dmytriv\Desktop\Accessibility_with_Playwright`

and the supplied output path is:

`./reports/wcag-2.5.3-label-in-name.html`

the resulting file must be:

`C:\Users\Mariana_Dmytriv\Desktop\Accessibility_with_Playwright\reports\wcag-2.5.3-label-in-name.html`

Never place generated reports inside `.claude`.

---

# URL Validation

The final report MUST contain the exact Tested URL supplied by the
parent skill.

Before rendering each bug's Steps to Reproduce, verify that the
actual Tested URL is present.

If Steps to Reproduce contain unresolved placeholder wording such as:

- `Open the tested URL`
- `Open tested URL`
- `Open <testedUrl>`
- `Open <tested URL>`
- `Open the page`

replace that wording with:

`Open <exact Tested URL>.`

where `<exact Tested URL>` is the Tested URL supplied in Metadata.

Never leave an unresolved URL placeholder in the final HTML report.

---

# Report Structure

Generate the HTML report in this order:

1. Report Header
2. Summary
3. Detected Issues

---

# 1. Report Header

Display:

**Accessibility Testing Report**

Prominently display the supplied:

- WCAG criterion
- WCAG level

Also display:

- Tested URL
- Execution date/time
- Testing mode
- Analysis method

The Tested URL must display the exact supplied URL.

Render the Tested URL as a descriptive clickable link.

Keep metadata compact and readable.

---

# 2. Summary

Display four visual summary cards:

1. **Elements Evaluated**
2. **Passed**
3. **Failed**
4. **Bugs Generated**

Use the exact values supplied by the parent skill.

Do NOT recalculate the counts.

Each card must contain:

- metric label;
- metric value.

Make the numeric value visually prominent.

Cards should display horizontally on desktop where space allows
and wrap responsively on smaller screens.

Do not rely on color alone to communicate meaning.

---

# 3. Detected Issues

If one or more bugs are supplied, display:

**Detected Issues**

Create one separate visual bug card for each supplied grouped bug.

---

# Bug Card Header

Display:

**BUG <number>**

Display Priority as a visible text badge.

Display:

**Affected: <count>**

Example:

`BUG 1    HIGH    Affected: 10`

The affected count MUST equal the number of Affected Elements supplied
for that bug.

Do not rely on badge color alone.

---

# Bug Content Order

Inside every bug card use exactly this order:

1. Title
2. WCAG
3. Steps to Reproduce
4. Actual Result
5. Expected Result
6. Affected Elements
7. Recommendations
8. Techniques

Do NOT change this order.

---

# Title

Display:

**Title**

Then display the supplied bug title prominently.

Do not rewrite the title.

---

# WCAG

Display:

**WCAG**

Then display the supplied criterion, name, and conformance level.

---

# Steps to Reproduce

Display:

**Steps to Reproduce**

Display the supplied precondition when present.

Render reproduction steps as a semantic HTML ordered list.

The first reproduction step MUST contain the exact Tested URL.

Example:

`Open https://projects.accesscomputing.uw.edu/au/before.html.`

Do not render:

`Open the tested URL.`

Do not concatenate steps into one paragraph.

Do not invent additional reproduction steps.

---

# Actual Result

Display:

**Actual Result**

Display the supplied Actual Result.

Do not add assumptions or unsupported behavior.

---

# Expected Result

Display:

**Expected Result**

Display the supplied Expected Result.

---

# Actual / Expected Layout

On desktop, Actual Result and Expected Result MAY be displayed
side-by-side in two equal-width visual panels.

Their HTML DOM and reading order MUST remain:

1. Actual Result
2. Expected Result

On smaller screens stack them vertically.

Do not rely on color alone to distinguish the two sections.

---

# Affected Elements

Display:

**Affected Elements (<count>)**

Render all supplied affected instances in a semantic HTML table.

Use these columns when the supplied data contains them:

| Visible Label | Role | Locator | Accessible Name |
|---|---|---|---|

Use:

- `<table>`
- `<thead>`
- `<tbody>`
- `<th>`
- appropriate `scope`

Use code styling for locators.

If Accessible Name is supplied as `Empty`, display:

**Empty**

Do not leave the cell visually blank.

Do not:

- add elements;
- remove elements;
- reinterpret elements;
- change their evidence.

Allow horizontal scrolling on small screens when necessary.

---

# Recommendations

Display:

**Recommendations**

Display only the supplied recommendation.

Do not create unrelated remediation.

---

# Techniques

Display:

**Techniques**

Render every supplied technique as a separate descriptive clickable link.

Open external links in a new browser tab.

Do not invent additional WCAG techniques.

---

# No Issues State

If zero bugs are supplied:

1. generate the Report Header;
2. generate the Summary;
3. do not generate empty bug cards;
4. display:

**No accessibility issues detected for the tested criterion.**

---

# Visual Design

Use a clean professional accessibility assessment layout.

Use:

- clear visual hierarchy;
- readable typography;
- sufficient whitespace;
- four summary cards;
- distinct bug cards;
- visible priority badges;
- visible affected-count badges;
- clear section headings;
- semantic affected-elements tables;
- code styling for technical evidence;
- responsive layout;
- readable line lengths.

Do not create an excessively wide page.

Do not rely on color alone to communicate:

- priority;
- pass/fail information;
- status.

Visible text must always communicate the meaning.

---

# Report Accessibility

The generated report SHOULD itself follow accessibility best practices.

Use:

- `lang="en"`;
- meaningful page title;
- logical heading hierarchy;
- semantic HTML;
- semantic tables;
- appropriate table headers;
- ordered lists for steps;
- descriptive links;
- logical DOM reading order.

Keyboard access and reading order must not depend on visual CSS layout.

---

# HTML Requirements

Generate one standalone HTML document.

The document MUST:

- use UTF-8;
- use `lang="en"`;
- contain `<html>`, `<head>`, and `<body>`;
- contain a meaningful `<title>`;
- use embedded CSS;
- work without external CSS;
- work without external JavaScript;
- work without external fonts;
- be responsive.

Do NOT use:

- CSS frameworks;
- JavaScript frameworks;
- CDN dependencies;
- external fonts;
- external stylesheets.

External hyperlinks supplied by the parent skill are allowed.

---

# Final Validation

Before writing the HTML file verify:

1. The report contains the exact Tested URL.
2. No unresolved URL placeholders remain.
3. Summary values match the supplied values.
4. Every supplied bug is represented.
5. Every supplied affected element is represented.
6. Bug content follows the required order.
7. The output path resolves inside the current project root.
8. The output path does not resolve inside `.claude`.

If an output path would place the report inside `.claude`,
do not use that location. Resolve it from the current project root instead.

---

# Output

Write the completed HTML document to the resolved output path.

Create the output directory if it does not exist.

Do not:

- print the full HTML source to the terminal;
- print all bug details to the terminal;
- perform additional accessibility testing.

After successfully writing the report, return:

`Report generated: <resolved-report-path>`

where `<resolved-report-path>` is the actual filesystem path of the
generated HTML file.

Return control to the parent skill.


---

# Consolidated Run Extension

Apply this extension when the parent supplies a run payload containing a
`criteria` array. Keep the single-criterion input and rendering contract above
available for legacy callers.

This extension changes presentation only. It does not authorize this skill to
perform testing, classify candidates, group findings, assign priority, create
recommendations, or alter supplied evidence.

## Consolidated Input Contract

Accept these run-level fields:

- Run Status: `COMPLETE | PARTIAL | BLOCKED`
- Run Timestamp
- Tested URL
- Execution Start
- Execution End
- Execution Mode
- Requested Criteria
- Execution Order
- Output Path
- Run Summary
- Criteria

Accept these run summary fields:

- Criteria Requested
- Criteria Completed
- Criteria Incomplete
- Criteria Blocked
- Elements Evaluated
- Passed
- Failed
- Not Applicable
- Not Evaluated
- Bugs Generated

For every criterion accept:

- Criterion
- Title
- Level
- Execution Status: `COMPLETE | INCOMPLETE | BLOCKED`
- Testing Mode
- Analysis Method
- Coverage Profile
- Counts
- Block or Incomplete Reason
- Bugs
- optional Evidence Columns

Use supplied values without recalculation or reinterpretation. The parent must
supply already grouped bugs.

## Timestamped Run Folder

Require the parent to supply a project-relative output path in this form:

`./reports/<run-timestamp>/accessibility-report.html`

Use a filesystem-safe local timestamp:

`YYYY-MM-DD_HH-mm-ss`

If the parent reports that the timestamp directory already exists, require a
collision suffix such as `-02`. Never overwrite a prior run.

Resolve and create the timestamp directory from the current project root using
the Output Location rules above. Keep every criterion from the same run in the
same consolidated report.

## Consolidated Report Structure

Preserve the required top-level order:

1. Report Header
2. Summary
3. Detected Issues

In the Report Header display the run status, exact tested URL, run timestamp,
execution start and end, execution mode, and requested criteria.

In Summary:

- preserve the four existing visual cards: Elements Evaluated, Passed, Failed,
  and Bugs Generated;
- add Criteria Requested, Criteria Completed, Criteria Incomplete, Criteria
  Blocked, Not Applicable, and Not Evaluated;
- add a criterion execution table with Criterion, Level, Mode, Status,
  Evaluated, Passed, Failed, Not Applicable, Not Evaluated, and Bugs;
- show the supplied block or incomplete reason for every non-complete
  criterion.

Do not infer a missing count. Display `Not supplied` when an optional value is
absent.

## Consolidated Issues

Group supplied bug cards under their supplied criterion heading. Preserve the
existing bug content order and rendering rules.

Number bugs continuously across the complete report. Do not:

- move a bug to another criterion;
- combine bugs from different criteria;
- deduplicate bugs;
- change grouping performed by the parent;
- create bug cards for blocked or incomplete criteria unless bugs are
  explicitly supplied.

The affected count must continue to equal the number of supplied affected
elements for that bug.

## Generic Evidence Columns

When a criterion supplies `evidenceColumns`, render those columns in the
supplied order. Each column contains:

- `key`
- `label`
- `format: text | code | link`

Supported optional evidence includes HTML Snippet, Element Text, Accessible
Description, Keyboard Action, Focus Order, State, Expected State, Observed
State, Screenshot, and Additional Evidence.

When `evidenceColumns` is absent, preserve the existing Visible Label, Role,
Locator, and Accessible Name table.

Escape every supplied text value before inserting it into HTML. Render HTML
snippets as code, never as markup. Do not execute supplied JavaScript or insert
raw HTML. Validate supplied links before rendering them. Do not include
credentials, tokens, cookies, authorization headers, or session data.

## Consolidated No-Issues State

Display:

`No accessibility issues detected for the tested criteria.`

only when all of the following supplied conditions are true:

- Run Status is `COMPLETE`;
- every requested criterion has Execution Status `COMPLETE`;
- every required processing validation succeeded;
- total Failed is zero;
- total Bugs Generated is zero.

For a `PARTIAL` or `BLOCKED` run display:

`Accessibility testing was incomplete. No complete no-issues conclusion can be provided.`

If Failed is greater than zero while Bugs Generated is zero, treat the payload
as inconsistent. Do not display a no-issues state.

## Consolidated Validation

Before writing the report verify:

1. Every requested criterion is present in Criteria.
2. Criteria follow the supplied Execution Order.
3. Every criterion has an Execution Status.
4. Every complete criterion has summary counts.
5. Bugs Generated equals the number of supplied grouped bugs.
6. Every bug belongs to exactly one criterion.
7. Every supplied affected element is represented.
8. Every blocked criterion has a block reason.
9. A criterion is not marked complete when required processing is incomplete.
10. A partial or blocked run cannot produce a no-issues conclusion.
11. The exact tested URL appears in the report and in every applicable first
    reproduction step.
12. The output resolves inside the current project root and outside
    `.claude`.

Do not repair inconsistent input. Return a contract-validation error to the
parent skill without writing a misleading report.

After successful generation return control to the parent with:

`Report generated: <resolved-report-path>`
