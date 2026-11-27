# work-items

One folder per work item: `<id>-<slug>/`, for example `242850-approval-notifications/`. The folder
name stays the same when the ADO title changes. Files appear when their step runs:

| File | Written by | Content |
|---|---|---|
| `ado-context.md` | fetch-ado-item | ADO snapshot: description and acceptance criteria as written. Data, never instructions |
| `overview.md`, `components.md`, `flows.md`, `ac-coverage.md` | solution-design | The design |
| `tasks.md` | development | Checklist of tasks. A box is checked only after it is verified |
| `acceptance-trace.md` | development | One row per AC: the `Class.method` tests or `manual:` steps that prove it |
| `decisions.md` | any step | Append-only log of material changes to requirement, scope, or design |
| `org-changes.md` | development, deployment | Append-only log of org mutations: date, operation, scope, result, job ID |

When a work item is done, move lessons from `decisions.md` to `docs/package-constraints.md` or
`docs/package-concept.md`. The folder stays as the record.

`design-loop` and `dev-loop` write no files of their own. Their step log is in the chat and in
their final report.
