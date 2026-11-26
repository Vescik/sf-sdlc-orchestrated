---
name: deployment
description: Validate and deploy Salesforce source or destructive changes from force-app/ or manifest/ to the dev org, confirm each real deploy in chat, then report and log the result. Use whenever repository metadata must reach the org.
---

# Deployment

Pass `--target-org intDev --json` on every command. Docs, work item files, agent configuration, and
tooling never need a deployment.

1. **Scope.** Name the components (`--source-dir`, `--metadata`, or `--manifest`). Only this work
   item's changes.
2. **Tests.** Choose `NoTestRun`, `RunSpecifiedTests` with `--tests`, or `RunLocalTests`.
   `<TODO: team default per change type>`
3. **Dry run:** `sf project deploy start --dry-run`.
4. **Confirm.** Say that changes will deploy to `intDev`, name scope and test level, and wait for
   yes. One yes covers one invocation: deploy, quick deploy, redeploy, or destructive deploy. Dry
   run, retrieve, status, report, and cancel need no confirmation. A
   [delivery run](../deliver-design/SKILL.md) with the launcher grant skips it for deploy and quick
   deploy.
5. **Deploy:** `sf project deploy start`. Follow a pending job with
   `sf project deploy report --job-id <ID>`.
6. **On failure,** fix the cause and repeat the dry run. The same error twice: report and ask.
7. **Log** the result in the org change log (see [development](../development/SKILL.md)).

## Removals

Use `--manifest <package.xml> --post-destructive-changes <destructiveChangesPost.xml>`. For a Flow,
deactivate the active version first (a deploy of its own, with confirmation) and list every version
as `<FlowName>-<number>`.

## Report

```text
Deployment: NOT RUN | DRY RUN | SUCCEEDED | FAILED | PARTIAL | CANCELED | IN PROGRESS | BLOCKED
Org alias: <alias>
Scope: <manifest or logical/source components>
Test policy: NoTestRun | RunSpecifiedTests | RunLocalTests
Latest job ID: <ID or none>
Iterations: <count>
Result/failure summary: <bounded factual summary>
Org mutations: <none or bounded list with verification>
Org change log: <path or none>
Unverified: <remaining limits>
```

Never present pending work as a pass or a dry run as deployed.
