# 900001: Acceptance coverage

- Work item ID: 900001
- Design files: [Overview](overview.md) [Components](components.md) [Flows](flows.md) [AC coverage](ac-coverage.md)

| Criterion | Planned behavior / components | Planned verification | Unresolved gap | Status |
|---|---|---|---|---|
| AC-1 | On Account save, `Account_Review_Overdue_Before_Save` checks `Last_Review_Date__c`; if it is more than 365 days before today, `Review_Overdue__c` is checked. | Flow test or Apex test inserts/updates Accounts with dates older than 365 days and asserts true. | [UNVERIFIED] exact custom field API names and org test date setup. | Covered |
| AC-2 | On Account save, the before-save Flow leaves `Review_Overdue__c` unchecked for a date exactly 365 days before today or later. | Test boundaries at exactly 365 days and at today/future dates; assert false. | [UNVERIFIED] exact custom field API names. | Covered |
| AC-3 | On Account save, the before-save Flow checks `Review_Overdue__c` when `Last_Review_Date__c` is blank. | Insert/update Account with blank date and assert true. | [UNVERIFIED] exact custom field API names. | Covered |
| AC-4 | The after-save Flow creates one open Task with subject `Account review overdue` for the Account owner only on unchecked-to-checked transition and only if no matching open Task exists. | Test first transition, repeat transition, existing open Task, and subject/status/owner assertions. | [UNVERIFIED] the standard Task open-status values and exact AccountId/WhatId relationship in org metadata. | Covered |
| AC-5 | Collection processing in both Flows updates all 200 Accounts correctly and performs bulk-safe lookups and creates. | Bulk Flow or Apex test saves 200 Accounts in one transaction with mixed dates and asserts all checkbox values and no limits errors. | [UNVERIFIED] test framework and org limits. | Covered |
| AC-6 | An Account Manager permission set grants read/edit for `Last_Review_Date__c` and read for `Review_Overdue__c`; users outside the role receive no access to either field. | `System.runAs` test verifies grant and deny behavior and Permission record setup is manually verified against the Account Manager ApplicationRole. | [UNVERIFIED] existing permission set, Permission record, and ApplicationRole owner; custom field API names. | Open |
