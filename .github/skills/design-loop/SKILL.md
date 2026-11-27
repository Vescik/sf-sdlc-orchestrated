---
name: design-loop
description: Write the design of a chosen option in a loop - the whole design, or only the ACs the request names - then check structure, check against the domain skills, fix, until it is ready for human acceptance or the budget is spent. Run only as the design-loop agent.
user-invocable: false
---

# Design loop

Run this only as the `design-loop` agent. Any other agent: stop and tell the human to send
`@design-loop design work item <ID> with option <N>`. The rules in the `design-loop` agent apply to
every step.

Input: a work item ID, a request, and the option the human chose. Output: design files that pass
every check, and a report. The loop never accepts the design: only a human sets
`Status: accepted`. Budget: 3 write attempts per run.

Keep the log in chat, not in a file: you have no editor, and a log file would leave the tree
dirty. When a step ends, print one row of this table, and repeat the whole table in the report:

```markdown
| # | Step | Commit | Result | Details |
|---|---|---|---|---|
```

## Start

1. Find `work-items/<ID>-*/`. Check: `ado-context.md` exists, the request names the chosen option
   (or `overview.md` records it), the branch `work-item/<ID>-<slug>`, and an empty
   `git status --porcelain`. Any check fails: report it and stop. No option chosen: tell the human
   to get the options from `@designer design work item <ID>` (or `... AC-<n> of work item <ID>`)
   first.
2. **Scope.** Take the mode from the request. A request that fits neither mode, or both, stops
   with `NEEDS HUMAN`.

   | Mode | Request | ACs in scope |
   |---|---|---|
   | `full` | "design work item <ID>" | every AC in `ado-context.md` |
   | `part` | names ACs: "design AC-4 of work item <ID>" | the named ACs, plus ACs new in `ado-context.md` when the request says "new ACs" |

   - `part` needs the four design files. Without them: stop and tell the human to use `full`.
   - A named AC that is not in `ac-coverage.md` and is not new in `ado-context.md`: stop with
     `NEEDS HUMAN`.
   - Record the base commit (`git rev-parse HEAD`). In `part` mode, rows outside scope must be the
     same at the end as at the base.
   - Print the ACs in scope and the base commit before Write.

## Loop

1. **Write.** Start the `designer` subagent with: the work item ID, "Called by design-loop", the
   mode, the chosen option, the ACs in scope, and on a later attempt the exact findings from the
   last step. It writes or fixes the design files and commits. Check with `git log -1` that a new
   commit exists. No commit: count the attempt, record it, and repeat only with a reason the
   designer named.
2. **Structure.** Check the files yourself, by reading them:
   - all four files exist: `overview.md`, `components.md`, `flows.md`, `ac-coverage.md`, each
     with the header from [solution-design](../solution-design/SKILL.md);
   - `overview.md` has `Status: draft`, never `accepted`;
   - every acceptance criterion in `ado-context.md` has a row in `ac-coverage.md`, and the AC IDs
     follow the rules in [solution-design](../solution-design/SKILL.md#4-write): IDs of the base
     unchanged, a new AC on the next free ID, a removed AC as `Removed from source`;
   - every row has a planned verification, and every `Explicit no-change` or
     `Removed from source` row has a reason;
   - every `[UNVERIFIED]` fact is also listed under open questions in `overview.md`.
   - `part` mode, against the base (`git diff <base> -- work-items/<ID>-*/`): only rows of ACs in
     scope changed in `ac-coverage.md`, and only rows whose `AC` column holds an AC in scope
     changed in `components.md`. A design that was `accepted` at the base has a new
     `decisions.md` entry for the change.
   Any gap: record it and go to Write with it. All pass: go to Domain check.
3. **Domain check.** Start the `reviewer` subagent for the design check of the work item, with the
   mode and the ACs in scope. It applies the Design check section of each matching domain skill
   and checks requirement and package. Findings marked `blocking`: go to Write with them. Other
   findings go to the report. Same blocking finding as the previous attempt: stop.

## Stop when

- Structure and domain check pass with no blocking finding: `READY FOR ACCEPTANCE`.
- Everything passes except open questions only a human can answer (business rules, `[UNVERIFIED]`
  facts the org cannot settle): `OPEN QUESTIONS`, with the questions and their owners.
- The budget is spent: `FAILED`, with the open findings.
- The same blocking finding comes back twice in a row: `STUCK`, with the finding.
- The chosen option cannot work, a component in scope is shared with an AC outside it, or the
  designer needs a human decision: `NEEDS HUMAN`, with the question. Never switch to another
  option, and never widen the scope.

## Report

```text
Result: READY FOR ACCEPTANCE | OPEN QUESTIONS | FAILED | STUCK | NEEDS HUMAN
Work item: <ID>, branch <branch>, HEAD <short SHA>, base <short SHA>
Mode: full | part, ACs in scope: <IDs>
Option: <the chosen option>
Write attempts: <n> of 3
ACs: <n> rows, <n> Covered, <n> Explicit no-change, <n> Removed from source, <n> Open
Status: draft (was accepted at the base: <yes/no>)
Domain skills applied: <names, or none matched>
Open questions: <question, owner>
Review: <blocking fixed: n>, open findings: <list>
Not verified: <for example: org facts marked [UNVERIFIED]; ACs outside scope>
Log: <the table of every step>
```

On `READY FOR ACCEPTANCE`, end with the next step for the human: read the design, set
`Status: accepted` in `overview.md` (or say so in chat), then
`@dev-loop implement work item <ID>`, or in `part` mode
`@dev-loop implement <ACs in scope> of work item <ID>`.
