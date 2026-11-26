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

Write each step to `loop.md` when it ends: attempt number, step, job ID, result, details.

## Start

1. Find `work-items/<ID>-*/`. Check: a design with `Status: accepted` (or the human asked to
   implement it), the branch `work-item/<ID>-<slug>`, and an empty `git status --porcelain`. Any
   check fails: report it and stop.
2. Create or append to `loop.md` in the work item folder:

   ```markdown
   | # | Step | Job ID | Result | Details |
   |---|---|---|---|---|
   ```

## Loop

1. **Build.** Start the `developer` subagent with: the work item ID, "Called by dev-loop", and on a
   later attempt the exact failures from the last step. It implements or fixes, updates
   `tasks.md` and `acceptance-trace.md`, and commits. Check with `git log -1` that a new commit
   exists. No commit: count the attempt, record it, and repeat only with a reason the developer
   named.
2. **Validate.** Test classes: every class named in `acceptance-trace.md`, plus the test classes
   in `git diff --name-only origin/main...HEAD`. Run:
   `sf project deploy start --dry-run --source-dir force-app --test-level RunSpecifiedTests --tests <classes> --wait 30 --target-org intDev --json`.
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
5. **Regression.** Run the same dry run with `--test-level RunLocalTests --wait 60`. Sort each
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
Validation: <test level>, job ID <ID>, <passed or failures>
Regression: RunLocalTests, job ID <ID>, <passed / failures ours / failures not ours>
Trace: <n> ACs, <n> by Apex tests, <n> manual, <n> none
Manual steps: <the manual: steps, for the human after deploy>
Review: <blocking fixed: n>, open findings: <list>
Not verified: <for example: Flow changes have no Apex tests>
Log: work-items/<id>-<slug>/loop.md
```

On `PASS` or `PASS WITH BLOCKED TESTS`, end with the next step for the human:
`@developer quick deploy validation <job ID> for work item <ID>` (it asks before the deploy), then
`@git-agent push and open a draft PR for work item <ID>`.
