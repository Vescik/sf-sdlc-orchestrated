---
name: deliver-design
description: Take an accepted design to a draft PR - build, scripted delivery check, review, draft PR - with each station as a sub-agent and a fixed attempt budget. Run only as the sdlc-conductor agent.
user-invocable: false
---

# Deliver design

Input: `itemId=<ID>` with a complete design. This request counts as asking for its implementation.
No design: stop and name [solution-design](../solution-design/SKILL.md). Output: a draft PR with a
PASS delivery check, or a report that names the failing criteria.

## Stations

Start each station as a sub-agent with the `task` tool and its `agent_type`. Give it the work item
folder, the skill, the inputs, and the expected output. Stations read inputs from files.

| Step | Agent | Request | Done when |
|---|---|---|---|
| 1 Build | `developer` | [development](../development/SKILL.md) without `tasks.md`, otherwise [continue-development](../continue-development/SKILL.md). Write `acceptance-trace.md`, commit, deploy, commit `deploy-job-id` | Committed and deployed |
| 2 Check | `developer` | `node scripts/check-delivery.mjs --work-item <folder>` | `delivery-check.json` for HEAD |
| 3 Review | `reviewer` | Implementation and security review of the commits since `main`. Answer only, write nothing | Findings returned |
| 4 Draft PR | `git-agent` | Push the branch and open a draft PR with the check result, manual steps, decisions, and open findings | PR URL |

`acceptance-trace.md` has one row per criterion: `| AC-1 | CaseRouterTest.routesHigh |`. The second
cell holds only `Class.method` or `manual: <step>` entries, separated by commas.

## Loop

1. Check FAIL: return to Build with the failing details. Each attempt changes a cause they name.
2. Check PASS: Review.
3. A finding that breaks a criterion, security, or a package boundary: return to Build, then check
   and review again. Other findings go into the PR.
4. No blocking finding and PASS for HEAD: Draft PR.

Budget: 6 checks per run.

## Stop only when

- The budget is spent: report the failing criteria. No PR.
- The check is BLOCKED: tests outside the delivery fail. List them; do not fix them.
- The design must change: a package-owned definition, the meaning of a criterion, or a new object
  or integration. Return to solution-design.
- The org version differs from the repository during the sync: ask.
- An operation needs a confirmation, or a hook denies an action the run needs.

For any other gap, take the smallest reasonable choice and record it in `decisions.md`. This list
replaces the other ask rules of the development skills during the run.

## Rules

- Only the check script decides success, and only it writes `delivery-check.json`.
- Never report a station as done unless its sub-agent ran.
- Deploy and quick deploy need no question only when the launcher granted autonomy for `intDev`
  (`SF_AGENTIC_AUTONOMOUS_ORG` is set). Otherwise confirm each one. Destructive operations always
  need confirmation.
- In this run, the push of the work item branch and the draft PR need no further instruction.
  Merge stays with a human.

## Report

One line per station, checks used, final check status, and the PR URL or the failing criteria.
