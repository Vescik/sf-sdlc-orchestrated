---
name: development
description: First implementation of a work item that has a design but no tasks.md, or a small change without a design. If tasks.md exists, use continue-development.
---

# Development

## Small change without a design

Small means: no new object, field, or integration; no change to a package-owned definition; no
change to the meaning of an acceptance criterion. State the plan, implement, deploy if source
changed, and record the plan in the PR. Skip "Implement a design". Anything larger needs a design
first ([solution-design](../solution-design/SKILL.md)).

## Implement a design

1. **Read** the design, `decisions.md`, and `ado-context.md`.
2. **Plan** in `tasks.md`: a checklist of tasks you can verify one by one. Each names its outcome
   and AC, components, and verification.
3. **Implement** in `force-app/`. Check off a task in `tasks.md` only after it is verified.
4. **Record** material changes to requirements, scope, or design in `decisions.md` before dependent
   work continues. Append only: what changed, why, what it affects. Update the affected design rows.
5. **Trace** each acceptance criterion to its proof in `acceptance-trace.md` (see below).
6. **Deploy** with the [deployment](../deployment/SKILL.md) skill when `force-app/` or `manifest/`
   changed.
7. **Commit** with [git-workflow](../git-workflow/SKILL.md), with `acceptance-trace.md`.

## Salesforce and package rules

- Apply the Implementation section of each matching domain skill (table in the workspace
  instructions). Data steps they name become `manual:` entries in `acceptance-trace.md`.
- Apex follows `apex.instructions.md`; Flows follow `flows.instructions.md`. Tie tests to the
  acceptance criteria: outcome assertions for Apex; a scenario, bulk, and fault test plan for Flows.
- When you create or change a component with a top-level `description` for a User Story or Bug, add
  `User Story <ID>: <title>` or `Bug <ID>: <title>` from `ado-context.md` once and keep earlier
  text. For a Flow, change only the main description. Add nothing to a ticketless change.
- Do not invent API names, picklist values, or package behavior. If the package behaves
  differently from the design, record it in `decisions.md` before changing code, and propose a
  `docs/package-constraints.md` entry.
- Remove any trace flags and debug levels you create for log analysis.

## Acceptance trace

`acceptance-trace.md` in the work item links every AC in `ac-coverage.md` to the test that proves
it. `ac-coverage.md` is the plan; the trace names the tests that exist.

```markdown
| AC | Covered by | Note |
|---|---|---|
| AC-1 | CaseRouterTest.routesHighToTier2 | |
| AC-2 | CaseRouterTest.routesOtherToTier1, CaseRouterTriggerTest.bulk200 | bulk case |
| AC-3 | manual: open a Case and check the Routing field | UI only |
```

- One row per AC, with the IDs from `ac-coverage.md`. An `Explicit no-change` AC gets
  `none: <reason>`.
- `Covered by` holds only `Class.method` entries, `manual: <step>`, or `none: <reason>`,
  separated by commas. Explanations go in `Note`.
- Each `Class.method` exists in `force-app/` and asserts the outcome the AC describes.
- Use `manual:` only when an Apex test cannot observe the behavior, such as page layout or UI-only
  behavior. A Flow AC needs an Apex test that runs the Flow through DML and asserts its effect,
  or a `manual:` step.
- Update the trace whenever a test or an AC changes.

## Org change log

After each org mutation returns a result, append one line to the work item's `org-changes.md` (no
work item: `docs/org-changes.md`): date, operation, org, scope, result, job ID. No credentials or
record values. Single test-record creates and updates get one summary line per session.

## Done

Tests pass, tasks are checked off, `decisions.md` is complete, the deployment report is given, and
the work is committed. Report what you could not verify as unverified.
