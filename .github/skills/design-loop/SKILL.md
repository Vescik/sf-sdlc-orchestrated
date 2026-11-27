---
name: design-loop
description: Write the design of a chosen option in a loop - write, check structure, check against the domain skills, fix - until it is ready for human acceptance or the budget is spent. Run only as the design-loop agent.
user-invocable: false
---

# Design loop

Run this only as the `design-loop` agent. Any other agent: stop and tell the human to send
`@design-loop design work item <ID> with option <N>`. The rules in the `design-loop` agent apply to
every step.

Input: a work item ID and the option the human chose. Output: design files that pass every check,
and a report. The loop never accepts the design: only a human sets `Status: accepted`. Budget: 3
write attempts per run.

Write each step to `design-loop.md` when it ends: attempt number, step, commit, result, details.

## Start

1. Find `work-items/<ID>-*/`. Check: `ado-context.md` exists, the request names the chosen option
   (or `overview.md` records it), the branch `work-item/<ID>-<slug>`, and an empty
   `git status --porcelain`. Any check fails: report it and stop. No option chosen: tell the human
   to get the options from `@designer design work item <ID>` first.
2. Create or append to `design-loop.md` in the work item folder:

   ```markdown
   | # | Step | Commit | Result | Details |
   |---|---|---|---|---|
   ```

## Loop

1. **Write.** Start the `designer` subagent with: the work item ID, "Called by design-loop", the
   chosen option, and on a later attempt the exact findings from the last step. It writes or fixes
   the design files and commits. Check with `git log -1` that a new commit exists. No commit:
   count the attempt, record it, and repeat only with a reason the designer named.
2. **Structure.** Check the files yourself, by reading them:
   - all four files exist: `overview.md`, `components.md`, `flows.md`, `ac-coverage.md`, each
     with the header from [solution-design](../solution-design/SKILL.md);
   - `overview.md` has `Status: draft`, never `accepted`;
   - every acceptance criterion in `ado-context.md` has a row in `ac-coverage.md`, in source
     order, labelled `AC-1`, `AC-2`, with the IDs of an earlier attempt unchanged;
   - every row has a planned verification, and every `Explicit no-change` row has a reason;
   - every `[UNVERIFIED]` fact is also listed under open questions in `overview.md`.
   Any gap: record it and go to Write with it. All pass: go to Domain check.
3. **Domain check.** Start the `reviewer` subagent for the design check of the work item. It
   applies the Design check section of each matching domain skill and checks requirement and
   package. Findings marked `blocking`: go to Write with them. Other findings go to the report.
   Same blocking finding as the previous attempt: stop.

## Stop when

- Structure and domain check pass with no blocking finding: `READY FOR ACCEPTANCE`.
- Everything passes except open questions only a human can answer (business rules, `[UNVERIFIED]`
  facts the org cannot settle): `OPEN QUESTIONS`, with the questions and their owners.
- The budget is spent: `FAILED`, with the open findings.
- The same blocking finding comes back twice in a row: `STUCK`, with the finding.
- The chosen option cannot work, or the designer needs a human decision: `NEEDS HUMAN`, with the
  question. Never switch to another option.

## Report

```text
Result: READY FOR ACCEPTANCE | OPEN QUESTIONS | FAILED | STUCK | NEEDS HUMAN
Work item: <ID>, branch <branch>, HEAD <short SHA>
Option: <the chosen option>
Write attempts: <n> of 3
ACs: <n> rows, <n> Covered, <n> Explicit no-change, <n> Open
Domain skills applied: <names, or none matched>
Open questions: <question, owner>
Review: <blocking fixed: n>, open findings: <list>
Not verified: <for example: org facts marked [UNVERIFIED]>
Log: work-items/<id>-<slug>/design-loop.md
```

On `READY FOR ACCEPTANCE`, end with the next step for the human: read the design, set
`Status: accepted` in `overview.md` (or say so in chat), then
`@dev-loop implement work item <ID>`.
