# Design details: components, flows, and decision history

## Component table (`components.md`)

```markdown
| Component / type (owner) | Planned change | Responsibility and behavior | Interactions | AC |
|---|---|---|---|---|
```

- Identify Salesforce components as `<Metadata type>:<full name>`. Mark new names as planned. One
  logical component may span several files.
- Owner is Subscriber, Package, or Platform. Mark unresolved ownership as such. Keep package impact
  explicit, including read-only dependencies.
- Planned change is `Create`, `Modify`, `Remove`, or `Read dependency only`. A rename is `Remove`
  plus `Create`.
- `AC` holds references only; the coverage table maps requirements to verification.
- For a nontrivial component, add a short contract after the table: trigger, inputs and outputs,
  permissions, errors, constraints.

## Diagram (`flows.md`)

Add at most one Mermaid diagram, only when it clarifies an interaction or a material boundary. Use
the component names from the table and mark uncertain links.

## Decision history (`decisions.md`)

Append an entry when a requirement or acceptance criterion changes materially, or the solution
departs from the design. Record the previous understanding, the new direction, source and reason,
what is affected, and blocked work. Append only. A Feature correction follows
[feature-context](feature-context.md). An entry does not trigger discovery, review, or QA reruns.
