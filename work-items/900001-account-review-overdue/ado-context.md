# 900001: Flag accounts with an overdue annual review

- Type: User Story
- State: Active
- Source: dummy test item, not in ADO. Do not fetch or refresh it.
- Retrieved: 2026-11-27T09:00:00Z
- Parent: none

## Description

As an Account Manager, I want accounts whose annual review is overdue to be flagged and to get a
follow-up task, so that no client goes more than a year without a review.

Today the review date is tracked in a spreadsheet. We want it on the Account record.

- Add a date field "Last Review Date" on Account. Account Managers fill it in after each review.
- Add a checkbox "Review Overdue" on Account. Users must not edit it; the system sets it when the
  account is saved.
- When an account becomes overdue, create a follow-up task for the account owner.

Data loads can update up to 200 accounts at a time.

## Acceptance criteria

1. When an account is saved and its Last Review Date is more than 365 days before today, Review
   Overdue is checked.
2. When an account is saved and its Last Review Date is 365 days before today or later, Review
   Overdue is unchecked.
3. When an account is saved and Last Review Date is empty, Review Overdue is checked.
4. When Review Overdue changes from unchecked to checked, one open Task with the subject "Account
   review overdue" is created for the account owner. If such an open Task already exists for the
   account, no second Task is created.
5. Saving 200 accounts in one transaction sets Review Overdue correctly on all of them without
   errors.
6. Users with the ApplicationRole "Account Manager" can read and edit Last Review Date and can read
   Review Overdue. Users without that ApplicationRole get no access to either field.
