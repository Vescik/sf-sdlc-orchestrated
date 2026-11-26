---
name: continue-development
description: Continue or change the implementation of a work item that already has tasks.md and a design, including in a new session (open tasks, follow-up changes, fixes, removals such as "remove this flow"). Not for the first implementation or for questions only.
---

# Continue development

Use this when the work item has `tasks.md` and a design, and the request changes or continues
source. Without `tasks.md`, use [development](../development/SKILL.md). From it, apply only:
Salesforce and package rules, Acceptance trace, Org change log, and Done.

The human's instruction is the decision, including a named removal or deactivation. Offer no
options. Deployment confirmation still applies. Follow the steps in order.

1. **Work item.** Take the ID from the request or the branch name and find `work-items/<ID>-*/`.
   No single match: ask.
2. **Context.** Read `tasks.md`, `decisions.md`, the design, and the latest lines in
   `org-changes.md`, if present. Open items in the log are history, not tasks.
3. **Sync.** No deployment recorded in `org-changes.md`: skip this step.
   1. Run `git status --porcelain -- force-app`. Not empty: show it and ask. Do not retrieve.
   2. Retrieve the item's components that exist in `force-app/` (`components.md` Create or Modify,
      and `org-changes.md`):
      `sf project retrieve start --metadata <Type:Name> --target-org intDev --json`
      Command error: report it and stop. Component not returned: say so and continue.
   3. Run the status command again.
      - Empty: continue.
      - No new files and `git diff -w` shows no changed lines: `git restore -- force-app`,
        continue.
      - Anything else: show the diff and ask: take the org version (commit it) or keep the
        repository version (discard the retrieved changes). Wait.
4. **Plan.** The work is the human's instruction; without one, the open tasks in `tasks.md`. From
   source, list path and Create, Modify, or Delete. Ask only if the work is ambiguous.
5. **Edit.** Apply the list. Removal: remove references, delete the file, add the destructive
   manifest. Add one `decisions.md` entry and update the affected design rows.
6. **Deploy** with the [deployment](../deployment/SKILL.md) skill.
7. **Record.** Update `tasks.md` and `acceptance-trace.md` (see
   [development](../development/SKILL.md#acceptance-trace)), append the `org-changes.md` line,
   commit, and report.

## Org reads after the sync

Only for: a component or field missing from `force-app/`; Flow versions and active state before a
removal or activation; a dry run or deployment error. Two reads without an answer: ask.

## Use solution-design instead if

the change touches a package-owned definition, changes the meaning of an acceptance criterion, or
adds a new object or integration. See [solution-design](../solution-design/SKILL.md).
