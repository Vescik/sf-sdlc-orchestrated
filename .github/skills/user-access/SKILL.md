---
name: user-access
description: Decide, build, and check user access for a change - which permission sets and Permission records give users access to new or changed objects, fields, Apex entry points, LWCs, tabs, record types, and custom permissions. Use in design, implementation, and review whenever a change adds or changes something users must reach.
---

# User access

## How access works here

Users never get a permission set directly. The chain is:

```text
User -> Resource -> ApplicationRole -> Permission (record) -> Salesforce permission set
```

- Each user has their own Resource. A Resource has an ApplicationRole.
- An ApplicationRole has many Permissions. A Permission is a custom object record that maps to one
  Salesforce permission set.
- The permission set reaches the user through the ApplicationRole.

So there are two kinds of change:

| Change | Kind | How it reaches the org |
|---|---|---|
| Object, field, class, tab, record type, or custom permission access in a permission set | Metadata in `force-app/main/default/permissionsets/` | Deploy |
| A new Permission record, or a Permission added to an ApplicationRole | Data | A human, after deploy. Never a deploy |

Never assign a permission set to a user with `PermissionSetAssignment` or `sf org assign permset`
outside an Apex test. The ApplicationRole owns the assignment.

## When this applies

The change adds or changes any of: an object, a field, an Apex class called from UI, Flow, or API,
an LWC or Aura component, a tab, a record type, a custom permission, or a Visualforce page.

## Design

For each component above, decide and write down:

1. **Who.** The ApplicationRoles whose users need it, and the access level (read, edit, create,
   delete, run). From the requirement; if it does not say, ask. Never grant to everyone by default.
2. **Which permission set.** An existing permission set that already serves those roles, or a new
   one. Prefer extending an existing one. A new one needs a new Permission record.
3. **Record in the design:**
   - `components.md`: one row per permission set, Planned change `Create` or `Modify`, listing the
     access it adds.
   - `components.md`: one row per new Permission record or ApplicationRole link, Planned change
     `Create`, owner Subscriber, marked as data.
   - `ac-coverage.md`: an AC for the access when the requirement names who must or must not have
     it.

## Design check

Checked in the design files. A failed item is `blocking`.

1. Every new or changed component in `components.md` from "When this applies" has a permission
   set row that lists its access, or a reason in `decisions.md` why it needs none.
2. Every permission set row names the ApplicationRoles it serves and the access level per
   component (read, edit, create, delete, run).
3. No access wider than the requirement: no grant to all users, no `View All` or `Modify All`
   unless the requirement asks for it.
4. Every new permission set has a row for its Permission record and the ApplicationRole link,
   marked as data, owner Subscriber.
5. When the requirement says who must or must not have access, `ac-coverage.md` has an AC row for
   it, verified by an Apex `System.runAs` test, and the Permission record as a `manual:` step.
6. A field set only by the system (formula, Flow, trigger) is planned read-only for users.

## Implementation

1. Edit the permission set files in `force-app/main/default/permissionsets/`. Grant only the access
   the design names. No `View All` or `Modify All` unless the design says so.
2. New field: add its field permission. New object: object permissions and its fields. New Apex
   class called from UI: class access. New tab: tab visibility.
3. Test the permission set in Apex: create a test user, assign the permission set in the test (the
   only allowed direct assignment), and assert the access with `System.runAs`, for example with
   `Security.stripInaccessible` or a DML that must succeed or fail. Name the test in
   `acceptance-trace.md`.
4. Permission records and ApplicationRole links are data. Do not create them. Add each as a
   `manual:` step in `acceptance-trace.md` and a task in `tasks.md`:
   `manual: create Permission "<name>" for permission set <API name> and add it to ApplicationRole <name>`.
   In a dev-loop run this is not a stop: the step goes to the report and the PR.

## Review

- Every new or changed component from "When this applies" has access in a permission set, or a
  recorded reason in `decisions.md` why it needs none. Missing access is `blocking`.
- Access is no wider than the design: extra objects, `View All`, `Modify All`, or edit where read
  was planned is `blocking`.
- No direct permission set assignment outside Apex tests.
- Each new permission set has its Permission record listed as a `manual:` step.


