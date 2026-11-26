---
name: development
description: First implementation of a work item that has a design but no tasks.md, or a small change without a design. If tasks.md exists, use continue-development.
user-invocable: false
---

# Development

## Small change without a design

Small means: no new object, field, or integration; no change to a package-owned definition; no
change to the meaning of an acceptance criterion. State the plan, implement, deploy if source
changed, and record the plan in the PR. Skip "Implement a design". Anything larger needs a design
first.

## Implement a design

1. **Read** the selected design, `decisions.md`, and `ado-context.md`.
2. **Plan** in `tasks.md`: coherent tasks you can verify one by one. Each names its outcome and AC,
   components, verification, and delivery or rollback steps where they apply. Each task has one
   status location.
3. **Implement** in `force-app/`. Update `tasks.md` after each unit.
4. **Record** material changes to requirements, scope, or design in `decisions.md` before dependent
   work continues. Append only: what changed, why, what it affects. Update the affected design rows.
5. **Deploy** with the [deployment](../deployment/SKILL.md) skill when `force-app/` or `manifest/`
   changed.

## Salesforce and package rules

- Apex follows `apex.instructions.md`; Flows follow `flows.instructions.md`. Tie tests to the
  acceptance criteria: outcome assertions for Apex; a scenario, bulk, and fault test plan for Flows.
- When you create or change a component with a top-level `description` for a User Story or Bug, add
  `User Story <ID>: <title>` or `Bug <ID>: <title>` from `ado-context.md` once and keep earlier
  text. For a Flow, change only the main description. If it does not fit, use the short form without
  the title. Add nothing to a ticketless change.
- Do not invent API names, picklist values, or package behavior. If the package behaves
  differently from the design, record it before changing code, and propose a
  `docs/package-constraints.md` entry.
- Remove any trace flags and debug levels you create for log analysis.

## Org change log

After each org mutation returns a result, append one entry to the work item's `org-changes.md`
(prepared Feature: its folder; no item: `docs/org-changes/`): date, operation, org, scope, result,
job ID. No credentials or record values. Single test-record creates and updates get one summary
line per session. The log never blocks an operation.

## Done

Tests pass, tasks are checked off, `decisions.md` is complete, the deployment report is given, and
the PR is prepared per [git-workflow](../git-workflow/SKILL.md). Ask the Reviewer only on request
or once before the PR. Report what you could not verify as unverified.
