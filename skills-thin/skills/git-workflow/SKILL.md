---
name: git-workflow
description: Prepare branches, commit, sync, resolve conflicts, push, open and merge pull requests for work items. Use for any Git or GitHub operation.
---

# Git workflow

- **Branch:** `work-item/<id>-<slug>`, or `feature/<feature-id>-<slug>` for a prepared Feature. On
  `main` or an unrelated branch, prepare or resume the right one yourself and continue. On
  divergence or conflicts, hand over to the Git Agent.
- **Commit:** authors commit their own work with `[WI-<id>]`. Stage only the work item's files and
  preserve unrelated work.
- **Push, PR publication, merge:** an explicit instruction for each. A handoff carries none of them.
  A [delivery run](../deliver-design/SKILL.md) covers the push and the draft PR after a PASS check.
- **GitHub writes** go through the Git Agent and the guarded routes. No custom wrapper, raw API, or
  CLI fallback.
- **PR:** use the repository template, link the work item, and state what was and was not verified.
  Keep required reviews and checks.

## To copy from the current files

`<TODO: sync and conflict procedure, gh issue rules, rulesets, GitHub MCP routing, AB# linking,
model-issued APPROVE, read-only evidence workflows>`
