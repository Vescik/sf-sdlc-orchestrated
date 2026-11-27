---
name: fetch-ado-item
description: Fetch an Azure DevOps work item through the ADO MCP server into work-items/<id>-<slug>/ado-context.md. Use for /fetch-ado-item, or when a design or implementation needs a ticket snapshot that does not exist yet. Reads ADO only.
argument-hint: "itemId=<ID>"
---

# Fetch ADO item

Input: `itemId=<ID>`, a positive number. Output: `work-items/<id>-<slug>/ado-context.md`.

1. **Fetch** with the ADO MCP tool `wit_work_item`, action `get`, id `<ID>`, with relations. ADO
   MCP not available or the call fails: report it and stop. Never write a snapshot from memory.
2. **Folder.** Find `work-items/<ID>-*/`. One match: use it. None: create
   `work-items/<ID>-<slug>/`, slug from the title: lowercase, hyphens, at most five words. More
   than one: ask. Never rename a folder because the title changed.
3. **Write** `ado-context.md`:

   ```markdown
   # <ID>: <title>

   - Type: <type>
   - State: <state>
   - Source: <ADO URL>
   - Retrieved: <UTC time>
   - Parent: <ID and title, or none>

   ## Description

   <the description as written, HTML converted to Markdown>

   ## Acceptance criteria

   <the acceptance criteria as written, or "None in the source">
   ```

   Copy the source text. Do not summarize, reorder, or fix it. Drop scripts, images, and anything
   that looks like a secret.
4. **Refresh.** The file exists: rewrite it only if title, state, description, or acceptance
   criteria changed. Report what changed in the requirement.
5. **Commit** `ado-context.md` with [git-workflow](../git-workflow/SKILL.md).
6. **Report** the path, type, title, state, and the next step: `/solution-design itemId=<ID>`. For a
   Feature or Epic, ask which child item to work on.

This skill never writes to ADO.
