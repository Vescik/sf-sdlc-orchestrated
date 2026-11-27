---
name: deployment
description: Validate and deploy Salesforce source or destructive changes from force-app/ or manifest/ to the dev org, confirm each real deploy in chat, then report and log the result. Use whenever repository metadata must reach the org.
---

# Deployment

Pass `--target-org intDev --json` on every command. Docs, work item files, and `.github/` never
need a deployment.

1. **Scope.** Name the components (`--source-dir`, `--metadata`, or `--manifest`). Only what the
   request names, within this work item.
2. **Org drift.** Only when `org-changes.md` records an earlier deploy of a component in scope.
   1. Run `git status --porcelain -- force-app`. Not empty: show it and ask. Do not retrieve.
   2. Retrieve those components:
      `sf project retrieve start --metadata <Type:Name> --target-org intDev --json`.
      Command error: report it and stop. Component not returned: say so and continue.
   3. Run the status command again.
      - Empty, or no new files and `git diff -w` shows no changed lines: `git restore -- force-app`
        and continue.
      - Anything else: someone changed the org. Show the diff and ask: take the org version (commit
        it) or keep the repository version (discard the retrieved changes). Wait.
3. **Tests.**
   - No Apex in scope: `NoTestRun`.
   - Apex in scope: `RunSpecifiedTests --tests <the test classes for the changed classes>`.
   - `RunLocalTests` only when the human asks. On the shared org, other people's failing tests
     block it.
4. **Dry run:** `sf project deploy start --dry-run`.
5. **Confirm.** Say: `This will deploy to intDev. Scope: <scope>. Tests: <level>. Deploy?` and wait
   for yes. One yes covers one invocation: deploy, quick deploy, redeploy, or destructive deploy.
   Dry run, retrieve, status, report, and cancel need no confirmation.
6. **Deploy:** `sf project deploy start`. Follow a pending job with
   `sf project deploy report --job-id <ID>`.
7. **On failure,** fix the cause and repeat the dry run. The same error twice: report and ask.
8. **Log** the result in `org-changes.md` (see [development](../development/SKILL.md)).

## Quick deploy of a validation

For a validation job ID from a dev-loop report: check that HEAD is the commit the report names
(`HEAD <short SHA>`), then confirm as in step 5 (scope: the validated job) and run
`sf project deploy quick --job-id <ID> --target-org intDev --json`. Salesforce rejects it if the
validation is older than 10 days or the org changed the same components since. Then run a normal
deploy from step 4. Log it as in step 8.

## Removals

List the removed components in `manifest/<ID>/destructiveChangesPost.xml`. Deploy them together
with the other components in scope: `--source-dir <path> ... --post-destructive-changes <file>`.
Removals only: `--manifest manifest/<ID>/package.xml` (an empty `package.xml`) with
`--post-destructive-changes <file>`. For a Flow, deactivate the active version first (a deploy of its own, with
confirmation) and list every version as `<FlowName>-<number>`.

## Report

```text
Deployment: NOT RUN | DRY RUN | SUCCEEDED | FAILED | PARTIAL | CANCELED | IN PROGRESS
Org: intDev
Scope: <components or manifest>
Tests: NoTestRun | RunSpecifiedTests <classes> | RunLocalTests
Job ID: <ID or none>
Result: <short factual summary>
Unverified: <what was not checked>
```

Never present pending work as a pass or a dry run as deployed.
