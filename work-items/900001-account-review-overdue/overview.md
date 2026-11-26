# 900001: Flag accounts with an overdue annual review

- Work item ID: 900001
- Design files: [Overview](overview.md) [Components](components.md) [Flows](flows.md) [AC coverage](ac-coverage.md)

## Status

Status: draft

## Proposed solution

- Add `Last_Review_Date__c` and `Review_Overdue__c` to the Account object.
- Make `Review_Overdue__c` read-only to users through field permissions and page layout/configuration.
- Use a before-save record-triggered Flow on Account to set `Review_Overdue__c` for all saves, including bulk loads.
- Use an after-save record-triggered Flow on Account to create at most one open follow-up Task when the checkbox changes from false to true.
- The task subject is `Account review overdue`; it is assigned to the Account owner's User record.
- Use collection-level Flow elements with no SOQL or DML inside loops.
- Grant `Last_Review_Date__c` read/edit and `Review_Overdue__c` read to the Account Manager ApplicationRole through an existing or new permission set.
- For users outside that ApplicationRole, provide no field access to either custom field.

## Open questions

- [UNVERIFIED] Does an existing permission set already serve the Account Manager ApplicationRole, and what API name should be used? Owner: implementation reviewer.
- [UNVERIFIED] Does the org already provide a Permission record linking that role to a Salesforce permission set? Owner: implementation and org administrator.
- [UNVERIFIED] Is the standard Account object the in-scope Account target, and are custom field API names agreed? Owner: work item owner.

## Scope

In scope: two Account fields, one before-save Flow, one after-save Flow, task deduplication, bulk behavior, field security, and the Account Manager ApplicationRole.

Out of scope: changing package-owned metadata, migrating existing spreadsheet data, redesigning Task pages, and creating Permission records or assigning PermissionSetAssignment directly.

## Options and decisions

- Option selected: record-triggered Flow for both before-save calculation and after-save task creation. It keeps the rules declarative and provides a bulk-safe collection implementation.
- Rejected: Apex trigger. It is more explicit for bulk and fault handling but was not selected because Flow was requested by the human choice.
- Rejected: formula checkbox plus Apex task automation. This avoids an explicit save update but would require a formula and Apex, adding mixed implementation patterns.
- Decision: use one open Task lookup per changed Account collection, then create only missing Tasks. This prevents duplicate work in the after-save path while remaining bulk-safe.

## Package impact

No package impact. The plan changes only standard Salesforce Account and Task components plus subscriber metadata. Evidence: repository is empty under `force-app/`; `docs/package-concept.md` has no package object facts; and the work item targets standard Account behavior. This evidence does not prove package behavior for a future package-owned extension, so no package-owned metadata is proposed.

## Verification and rollback

- Verify AC-1 through AC-5 with Flow tests or Apex test coverage that proves before-save values, task transition/deduplication, empty dates, 200-record saves, and bulk task limits.
- Verify AC-6 with a permission-set test that uses `System.runAs` and tests read/edit for Account Managers and no access for users outside the role.
- Roll back by activating no replacement, deactivating both new Flow definitions/versions, and removing only the two subscriber fields and permission metadata through a reviewed removal manifest.
- Permission records and ApplicationRole links require manual rollback or correction; they are not deployed by source metadata.

## Requirement baseline

- ADO work item: [ado-context.md](ado-context.md)
- Source: dummy test item, not in ADO
- Retrieved: 2026-11-27T09:00:00Z

## Evidence and limits

Evidence is the provided dummy ADO snapshot and the repository source scan. No Salesforce org facts or existing permission metadata were available. Field names, permission-set reuse, and ApplicationRole data links remain unverified in an org-aware implementation step.
