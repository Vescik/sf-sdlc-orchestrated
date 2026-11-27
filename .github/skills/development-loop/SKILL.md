---
name: development-loop
description: Build and validate what the request names - the whole accepted design, part of it by AC or component, or named components and removals - in a loop of build, dry run, trace, review, and regression check, until it is green or the budget is spent. Run only as the dev-loop agent.
user-invocable: false
---

# Development loop

Run this only as the `dev-loop` agent. Any other agent: stop and tell the human to send
`@dev-loop implement work item <ID>`. The rules in the `dev-loop` agent apply to every step.

Input: a work item ID and a request. Output: a validated build of the request's scope on the work
item branch, a validation job ID for quick deploy, and a report. Budget: 5 build attempts per run.

Keep the log in chat, not in a file: you have no editor, and a log file would leave the tree
dirty. When a step ends, print one row of this table, and repeat the whole table in the report:

```markdown
| # | Step | Job ID | Result | Details |
|---|---|---|---|---|
```

## Start

1. Find `work-items/<ID>-*/`. Check: the branch `work-item/<ID>-<slug>` and an empty
   `git status --porcelain`. Any check fails: report it and stop.
2. **Scope.** Take the mode from the request. Do not guess one: a request that fits none, or more
   than one, stops with `NEEDS HUMAN`.

   | Mode | Request | Scope | ACs |
   |---|---|---|---|
   | `design` | "implement work item <ID>", "the whole design" | every `components.md` row with `Create`, `Modify`, or `Remove` | all in `ac-coverage.md` |
   | `part` | names ACs (`AC-2, AC-3`) or components (`ApexClass:Foo`) | the `components.md` rows with those ACs in their `AC` column, or with those names | the ACs the request names, or those in the `AC` column of the chosen rows |
   | `named` | names components to deploy or remove, outside the design or without one | exactly the named components, as `Type:Name` with `Create`, `Modify`, or `Remove` | none |

   - `design` and `part` need a design with `Status: accepted`, or a request from the human to
     implement it. `named` needs none.
   - A named AC or component that is not in `components.md` or `ac-coverage.md`: stop with
     `NEEDS HUMAN`.
   - Rows owned by Package, rows marked as data, and `Read dependency only` rows are never in
     scope. Data rows go to the report as manual steps.
   - A Flow with `Remove`: check that it has no active version with
     `sf data query --use-tooling-api --query "SELECT ActiveVersionId FROM FlowDefinition WHERE DeveloperName = '<name>'" --target-org intDev --json`.
     An active version cannot be deleted in the same deploy: stop with `NEEDS HUMAN`, and name the
     deactivation deploy in [deployment](../deployment/SKILL.md) as the next step.
   - Print the scope as a table (`Type:Name`, change, ACs) before Build. This table is the scope
     for every later step, the developer, and the reviewer.

## Loop

1. **Build.** Start the `developer` subagent with: the work item ID, "Called by dev-loop", the
   mode, the scope table, the ACs in scope, and on a later attempt the exact failures from the
   last step. It implements or fixes only the scope, updates `tasks.md` and `acceptance-trace.md`,
   and commits. Check with `git log -1` that a new commit exists. No commit: count the attempt,
   record it, and repeat only with a reason the developer named.
   - `named` mode, first attempt: when every component in scope already exists in `force-app/`
     with no change still to make, and the removals manifest exists, skip Build and say so.
2. **Check the diff.** Take `git diff --name-only origin/main...HEAD -- force-app manifest`.
   - A changed file that belongs to no component in scope: stop with `NEEDS HUMAN`.
   - A `Create` or `Modify` component in scope with no file in `force-app/`: go to Build with it.
   - A `Remove` component in scope that is still in `force-app/`, or missing from
     `manifest/<ID>/destructiveChangesPost.xml`: go to Build with it.
3. **Validate.** Pass each file of a `Create` or `Modify` component in scope as its own
   `--source-dir <path>`. With `Remove` components in scope, add
   `--post-destructive-changes manifest/<ID>/destructiveChangesPost.xml`; with removals only, use
   `--manifest manifest/<ID>/package.xml` (an empty `package.xml`) instead of `--source-dir`.
   Test classes: every class named in `acceptance-trace.md` for the ACs in scope, plus the test
   classes in scope. Run:
   `sf project deploy start --dry-run --source-dir <path> [--source-dir <path> ...] [--post-destructive-changes <file>] --test-level RunSpecifiedTests --tests <classes> --wait 30 --target-org intDev --json`.
   Check that the components in the result, deployed and deleted, are exactly the ones in scope.
   Any other component: stop with `NEEDS HUMAN`.
   No test class: use `--test-level NoTestRun` (compile only) and say so in the report.
   Timeout: follow with `sf project deploy report --job-id <ID> --wait 30 --target-org intDev --json`.
   - A failure because a component outside scope is missing in the org (a field, class, or object
     that another part of the design creates): stop with `NEEDS HUMAN: add <Type:Name> to the
     scope?`. Never send it to Build.
   - Other failures: record them and go to Build with them.
   - Same failure as the previous attempt, twice in a row: stop.
   - Passed: go to Trace.
4. **Trace.** `named` mode: skip and say so in the report. Otherwise check `acceptance-trace.md`
   for the ACs in scope only, against `ac-coverage.md` and the test results in the validation JSON
   (`runTestResult` successes and failures, by class and method name):
   - every AC in scope has a row;
   - every `Class.method` in those rows has a passing result in this validation;
   - `none:` only for an AC with status `Explicit no-change`.
   Any gap: record the AC and the gap, and go to Build with it. All pass: go to Review. A
   `manual:` entry never fails this check and never stops the loop: collect it for the report.
5. **Review.** Start the `reviewer` subagent for the implementation of the work item, with
   "Called by dev-loop", the scope table, and the ACs in scope. Findings marked `blocking`: go to
   Build with them. Other findings go to the report.
6. **Regression.** Run the same dry run, with the same scope, with
   `--test-level RunLocalTests --wait 60`. On `PASS`, its job ID is the one for quick deploy; with
   failures, use the last Validate job ID, which passed at the same HEAD. Sort each failed test:
   - **Ours:** the test class is in scope, or it references a component in scope (`grep` the
     class for the component names). Go to Build with it.
   - **Not ours:** neither. Do not fix it. Record it as `BLOCKED (outside scope)`.
   - Unsure: treat it as not ours and say why.
   All failures not ours and the rest green: the result is `PASS WITH BLOCKED TESTS`.

## Stop when

- Regression passes: `PASS`.
- The budget is spent: `FAILED`, with the open failures.
- The same failure comes back twice in a row: `STUCK`, with the failure.
- The scope is unclear or needs a component outside it, the design must change (new object or
  integration, package-owned definition, changed meaning of an AC), the org differs from the
  repository, or the developer needs a human decision: `NEEDS HUMAN`, with the question.

## Report

```text
Result: PASS | PASS WITH BLOCKED TESTS | FAILED | STUCK | NEEDS HUMAN
Work item: <ID>, branch <branch>, HEAD <short SHA>
Mode: design | part | named
Scope: <the scope table>
Build attempts: <n> of 5
Validation: <test level>, job ID <ID>, <passed or failures>
Regression: RunLocalTests, job ID <ID>, <passed / failures ours / failures not ours>
Trace: <n> ACs in scope, <n> by Apex tests, <n> manual, <n> none | skipped (named mode)
Manual steps: <the manual: steps and data rows, for the human after deploy>
Review: <blocking fixed: n>, open findings: <list>
Not verified: <for example: Flow changes have no Apex tests; ACs outside scope>
Log: <the table of every step>
```

On `PASS` or `PASS WITH BLOCKED TESTS`, end with the next step for the human:
`@developer quick deploy validation <job ID> for work item <ID>` (it asks before the deploy), then
`@git-agent push and open a draft PR for work item <ID>`.
