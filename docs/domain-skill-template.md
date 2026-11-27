# Domain skill template

Copy this to `.github/skills/<name>/SKILL.md`, replace the placeholders, and add a row to the
Domain skills table in `.github/copilot-instructions.md`. Keep every section, even if one says
"Nothing in this area".

Write the Design check and Review items so that a reader can pass or fail each one from the files
alone ("components.md has a row for X"), not by judgement ("the design is good"). `design-loop`
and `dev-loop` loop on them, and a vague item never passes or never fails.

````markdown
---
name: <name>
description: <What area this covers and the decisions it makes.> Use in design, implementation, and review whenever a change <trigger>.
---

# <Title>

## How it works here

<The org-specific facts: the objects, the chain, what is metadata and what is data.>

## When this applies

<The components or changes that trigger this skill. Same text as the trigger in the table.>

## Design

<What the designer decides and where it is written: which file, which rows.>

## Design check

Checked in the design files. A failed item is `blocking`.

1. <A rule the design files must pass.>

## Implementation

<How the developer builds it. Data that a human creates after deploy is a `manual:` step in
`acceptance-trace.md`, never a stop.>

## Review

<Rules the implementation must pass. Say which failures are `blocking`.>

## To confirm

<Open facts about this area, with an owner. Remove the section when it is empty.>
````
