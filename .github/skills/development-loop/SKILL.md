---
name: development-loop
description: Implement an accepted design in a loop - build, dry-run validate, trace, review, regression check - until it is green or the budget is spent. Run only as the dev-loop agent.
user-invocable: false
---

# Development loop

Run this only as the `dev-loop` agent. Any other agent: stop and tell the human to send
`@dev-loop implement work item <ID>`. The rules in the `dev-loop` agent apply to every step.

Input: a work item ID with an accepted design. Output: a validated build on the work item branch,
a validation job ID for quick deploy, and a report. Budget: 5 build attempts per run.

Keep the log in chat, not in a file: you have no editor, and a log file would leave the tree
dirty. When a step ends, print one row of this table, and repeat the whole table in the report:

```markdown
| # | Step | Job ID | Result | Details |
|---|---|---|---|---|
```

## Start

1. Find `work-items/<ID>-*/`. Check: a design with `Status: accepted` (or the human asked to
   implement it), the branch `work-item/<ID>-<slug>`, and an empty `git status --porcelain`. Any
   check fails: report it and stop.

## Loop

1. **Build.** Start the `developer` subagent with: the work item ID, "Called by dev-loop", and on a
   later attempt the exact failures from the last step. It implements or fixes, updates
   `tasks.md` and `acceptance-trace.md`, and commits. Check with `git log -1` that a new commit
   exists. No commit: count the attempt, record it, and repeat only with a reason the developer
   named.
2. **Validate.** Scope: only this work item's changes, never all of `force-app`. Take the files
   from `git diff --name-only --diff-filter=d origin/main...HEAD -- force-app` and pass each
   one as its own `--source-dir <path>`. Deleted files (`--diff-filter=D`) cannot be in a dry run:
   list them in the report as removals for the human, through
   [deployment](../deployment/SKILL.md). No file in scope: stop with `NEEDS HUMAN`.
   Test classes: every class named in `acceptance-trace.md`, plus the test classes in that diff.
   Run:
   `sf project deploy start --dry-run --source-dir <path> [--source-dir <path> ...] --test-level RunSpecifiedTests --tests <classes> --wait 30 --target-org intDev --json`.
   Check that the components in the result are the ones in the diff. Any other component: stop
   with `NEEDS HUMAN`.
   No test class in the trace or the diff: use `--test-level NoTestRun` (compile only) and say so
   in the report.
   Timeout: follow with `sf project deploy report --job-id <ID> --wait 30 --target-org intDev --json`.
   - Failures: record them and go to Build with them.
   - Same failure as the previous attempt, twice in a row: stop.
   - Passed: go to Trace.
3. **Trace.** Check `acceptance-trace.md` against `ac-coverage.md` and the test results in the
   validation JSON (`runTestResult` successes and failures, by class and method name):
   - every AC in `ac-coverage.md` has a row;
   - every `Class.method` in the trace has a passing result in this validation;
   - `none:` only for an AC with status `Explicit no-change`.
   Any gap: record the AC and the gap, and go to Build with it. All pass: go to Review. A
   `manual:` entry never fails this check and never stops the loop: collect it for the report.
4. **Review.** Start the `reviewer` subagent for the implementation of the work item. Findings
   marked `blocking`: go to Build with them. Other findings go to the report.
5. **Regression.** Run the same dry run, with the same `--source-dir` scope, with
   `--test-level RunLocalTests --wait 60`. On `PASS`, its job ID is the one for quick deploy; with
   failures, use the last Validate job ID, which passed at the same HEAD. Sort each
   failed test:
   - **Ours:** the test class is in the diff, or it references a component in the diff
     (`grep` the class for the component names). Go to Build with it.
   - **Not ours:** neither. Do not fix it. Record it as `BLOCKED (outside scope)`.
   - Unsure: treat it as not ours and say why.
   All failures not ours and the rest green: the result is `PASS WITH BLOCKED TESTS`.

## Stop when

- Regression passes: `PASS`.
- The budget is spent: `FAILED`, with the open failures.
- The same failure comes back twice in a row: `STUCK`, with the failure.
- The design must change (new object or integration, package-owned definition, changed meaning
  of an AC), the org differs from the repository, or the developer needs a human decision:
  `NEEDS HUMAN`, with the question.

## Report

```text
Result: PASS | PASS WITH BLOCKED TESTS | FAILED | STUCK | NEEDS HUMAN
Work item: <ID>, branch <branch>, HEAD <short SHA>
Build attempts: <n> of 5
Scope: <n> files from the diff; removals for the human: <paths, or none>
Validation: <test level>, job ID <ID>, <passed or failures>
Regression: RunLocalTests, job ID <ID>, <passed / failures ours / failures not ours>
Trace: <n> ACs, <n> by Apex tests, <n> manual, <n> none
Manual steps: <the manual: steps, for the human after deploy>
Review: <blocking fixed: n>, open findings: <list>
Not verified: <for example: Flow changes have no Apex tests>
Log: <the table of every step>
```

On `PASS` or `PASS WITH BLOCKED TESTS`, end with the next step for the human:
`@developer quick deploy validation <job ID> for work item <ID>` (it asks before the deploy), then
`@git-agent push and open a draft PR for work item <ID>`.
