# Prepared Feature delivery context

## Resolve membership from local files

1. Search active `work-items/*/delivery-map.md` for the exact numeric Story ID listed as
   `included`. Ignore delivered folders, `delivery-map.archived[-N].md`, and `deferred` listings.
2. No match: proceed standalone. An ADO parent relation alone activates nothing.
3. One match: check the map's Feature ID and source link against the Feature `ado-context.md`, read
   both, and use them as broader context.
4. More than one match: name the maps and ask which Feature is active. Do not pick by title, path,
   or date.

## Requirement baseline (one match only)

```text
Prepared delivery context:
- Feature: <feature ID and title>
- Feature context: work-items/<feature-id>-<slug>/ado-context.md
- Feature source link and retrieval time: <ADO URL and UTC timestamp>
- Delivery map: work-items/<feature-id>-<slug>/delivery-map.md
- Membership: included
```

The Story's acceptance criteria stay authoritative. Report a conflict with Feature text as a
finding; do not rewrite the Story context. The Story stays the unit of design and QA.

## Feature-level corrections

Record a Feature membership or scope correction in the Feature's `decisions.md`, and update only
the affected design, map, and task files.
