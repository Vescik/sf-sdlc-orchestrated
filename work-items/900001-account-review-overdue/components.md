# 900001: Components

- Work item ID: 900001
- Design files: [Overview](overview.md) [Components](components.md) [Flows](flows.md) [AC coverage](ac-coverage.md)

| Component / type (owner) | Planned change | Responsibility and behavior | Interactions | AC |
|---|---|---|---|---|
| `Object:Account` (Subscriber) | Modify | Add custom fields `Last_Review_Date__c` (Date) and `Review_Overdue__c` (Checkbox). | Platform record save; permission sets; both planned Flows. | AC-1, AC-2, AC-3, AC-6 |
| `Object:Task` (Platform) | Read dependency only | Find open Tasks by AccountId and subject; create follow-up Tasks with the Account owner. | After-save Flow. | AC-4 |
| `Flow:Account_Review_Overdue_Before_Save` (Subscriber) | Create | Before-save, calculate `Review_Overdue__c=true` when `Last_Review_Date__c` is blank or more than 365 days old; otherwise false. | Runs before Account save. | AC-1, AC-2, AC-3, AC-5 |
| `Flow:Account_Review_Overdue_After_Save` (Subscriber) | Create | After-save, identify false-to-true transitions, query open matching Tasks in bulk, and create only missing Tasks. | Reads changed Accounts and creates Tasks. | AC-4, AC-5 |
| `PermissionSet:<Account Manager existing or new name>` (Subscriber) | Create or Modify | Give read/edit on `Last_Review_Date__c` and read on `Review_Overdue__c`; no extra object or system permissions. | Field-level security for the Account Manager ApplicationRole. | AC-6 |
| `PermissionRecord:<name>` (Subscriber, data) | Create | Link the Account Manager ApplicationRole to the selected Salesforce permission set. | Manual org-data step, not source deployment. | AC-6 |
| `ApplicationRole:Account Manager` (Subscriber or Package, unresolved) | Read dependency only | Supplies users to the Permission record and therefore to the permission set. | Permission record and Salesforce permission set. | AC-6 |

## Contracts

### Account before-save Flow

- Trigger: after a record is saved by update or insert, before-save entry for the standard Account object.
- Inputs: `Last_Review_Date__c` and today.
- Output: `Review_Overdue__c` true or false.
- Boundary: calculation-only, no SOQL or DML; bulk-safe for 200 records.
- Error behavior: formula/Flow validation failures are surfaced through the normal save; no fault connectors are required for calculation-only metadata.

### Account after-save Flow

- Trigger: after-save on Account insert or update only when `Review_Overdue__c` changed from false to true.
- Inputs: changed Account Id, OwnerId, and prior/current checkbox values.
- Output: zero or one open Task per matching Account.
- Boundary: collection queries and create elements; no queries or DML inside loops.
- Dedupe rule: subject exactly `Account review overdue`, open Status, and matching WhatId/AccountId.
- Error behavior: Task create faults are observable and surfaced with the Account save outcome; no secret or record data is exposed.

### Field access

- Account Manager users receive read/edit for `Last_Review_Date__c` and read for `Review_Overdue__c`.
- Users outside that ApplicationRole receive no access to either field.
- The field permissions belong in a permission set; the role-to-permission link is manual data, not a direct user assignment.
