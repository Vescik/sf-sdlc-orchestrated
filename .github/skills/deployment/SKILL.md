---
name: deployment
description: Validate and deploy Salesforce source or destructive changes from force-app/ or manifest/ to the dev org, confirm each real deploy in chat, then report and log the result. Use whenever repository metadata must reach the org.
---

# Deployment

Pass `--target-org intDev --json` on every command. Docs, work item files, and `.github/` never
need a deployment.

1. **Scope.** Name the components (`--source-dir`, `--metadata`, or `--manifest`). Only this work
   item's changes.
2. **Tests.**
   - No Apex in scope: `NoTestRun`.
   - Apex in scope: `RunSpecifiedTests --tests <the test classes for the changed classes>`.
   - `RunLocalTests` only when the human asks. On the shared org, other people's failing tests
     block it.
3. **Dry run:** `sf project deploy start --dry-run`.
4. **Confirm.** Say: `This will deploy to intDev. Scope: <scope>. Tests: <level>. Deploy?` and wait
   for yes. One yes covers one invocation: deploy, quick deploy, redeploy, or destructive deploy.
   Dry run, retrieve, status, report, and cancel need no confirmation.
5. **Deploy:** `sf project deploy start`. Follow a pending job with
   `sf project deploy report --job-id <ID>`.
6. **On failure,** fix the cause and repeat the dry run. The same error twice: report and ask.
7. **Log** the result in `org-changes.md` (see [development](../development/SKILL.md)).

## Removals

Use `--manifest <package.xml> --post-destructive-changes <destructiveChangesPost.xml>` in
`manifest/`. For a Flow, deactivate the active version first (a deploy of its own, with
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
