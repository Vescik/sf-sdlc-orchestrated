---
name: git-workflow
description: Branches, commits, sync, conflicts, push, and pull requests for work items. Local Git through the terminal, pull requests through the GitHub MCP server. Use for any Git or GitHub operation.
---

# Git workflow

## Branch

`work-item/<id>-<slug>` for a work item, `chore/<short-description>` without one. On `main` or an
unrelated branch, create or switch to the right one yourself and continue. Never commit work item
changes to `main`.

## Commit

- Commit each coherent, verified result yourself; no separate request is needed.
- Stage only this work item's files (`git add <paths>`). Preserve unrelated work: never
  `git stash`, `reset --hard`, or `clean` to make a commit pass.
- Message subject: `[WI-<id>] <what changed> AB#<id>`, or `[chore] <what changed>` without a work
  item. English, imperative. No `Fixes`, `Resolves`, or `Closes`.
- Check the result with `git log -1 --stat` and report the SHA.

## Sync and conflicts

- `git fetch origin`, then `git rebase origin/main` on a branch only you use; `git merge origin/main`
  on a shared one.
- Resolve conflicts in files of this work item yourself. A conflict in someone else's change, or
  in a meaning you cannot decide: show it and ask.
- Never force push a shared branch, rewrite `main`, or delete remote branches. Force push on your
  own branch needs an explicit instruction.

## Push and pull request

These need an explicit instruction for each: push, PR creation or update, merge. A finished task
or a passing check is not an instruction. A next-step line the human sent counts as the
instruction for what it names, nothing more. A push instruction does not create a PR.

- **Push:** `git push -u origin <branch>`.
- **PR:** with the GitHub MCP tools, which only the Git Agent has. First search for an open PR from the same branch and update it
  instead of creating a second one. Create it as a draft unless the human asks for ready. Base
  `main`. Fill `.github/pull_request_template.md` with facts only: what ran, what did not.
- **Merge:** only on an explicit instruction naming the PR. Never approve a PR.
- Report the PR URL, its state, and its checks.

No `gh` CLI, raw API calls, or tokens for GitHub writes. If the GitHub MCP server is not
available, report it and give the human the push command and PR text instead.
