# sf-sdlc-orchestrated

A thin GitHub Copilot workspace for Salesforce work on top of the Kantata SX (`KimbleOne__`)
managed package. It takes an ADO work item to a design, an implementation, a deploy to `intDev`,
and a draft PR.

It has six skills, two MCP servers, and no scripts or hooks. The guardrails are the instructions
plus the VS Code, MCP, and GitHub configuration below.

## Contents

| Path | Purpose |
|---|---|
| `.github/copilot-instructions.md` | Routing, tools, org access, confirmations |
| `.github/skills/` | `fetch-ado-item`, `solution-design`, `development`, `continue-development`, `deployment`, `git-workflow` |
| `.github/instructions/` | Package boundaries, Apex rules, Flow rules |
| `.vscode/mcp.json` | `ado` (Azure DevOps) and `github` MCP servers |
| `.vscode/settings.json` | Skills on, read-only commands auto-approved, everything else asks |
| `work-items/` | One folder per work item. See its README |
| `docs/` | Package concept and constraints, filled by the team |
| `force-app/`, `manifest/` | Salesforce DX source |

## Flow

```text
/fetch-ado-item itemId=<ID>     ADO -> work-items/<id>-<slug>/ado-context.md
/solution-design itemId=<ID>    options -> your choice -> design files
"implement <ID>"                tasks.md -> force-app/ -> dry run -> your yes -> deploy
"push and open a PR for <ID>"   draft PR through GitHub MCP
```

## Setup

1. Install VS Code, the GitHub Copilot extension, the Salesforce CLI (`sf`), and Node.js (for the
   ADO MCP server through `npx`).
2. Log in to the dev org with the alias `intDev`:

   ```bash
   sf org login web --alias intDev --instance-url https://test.salesforce.com
   ```

3. Open the folder in VS Code. Start both servers from `.vscode/mcp.json`: enter the ADO
   organization name when asked, then sign in to Azure DevOps and GitHub in the browser.
4. In Copilot Chat, pick Agent mode and run `/fetch-ado-item itemId=<ID>`.

## Guardrails

These hold without any script. Keep them in place.

1. **No production login on developer machines.** `sf org list` shows only `intDev` and other
   sandboxes. An agent cannot deploy to an org the CLI cannot reach.
2. **Approval clicks stay on.** `.vscode/settings.json` auto-approves only read-only `git`, `sf`
   reads, and dry runs against `intDev`. A real deploy, data change, or push always asks. Do not
   turn on `chat.tools.global.autoApprove` and do not add `"sf": true`.
3. **MCP tools are not auto-approved.** Each ADO or GitHub MCP call asks in VS Code. Approve reads;
   read any write before you approve it.
4. **Branch protection on `main`.** In GitHub, add a ruleset for `main`: require a pull request with
   one approval, and block force pushes. Then a PR cannot be merged without a second person, even
   through the MCP server.
5. **Narrow MCP scope.** ADO loads only the `core`, `work`, and `work-items` domains. GitHub loads
   only the `context`, `repos`, and `pull_requests` toolsets.
6. **Pin the ADO MCP version** after the first successful run: replace `@azure-devops/mcp` with
   `@azure-devops/mcp@<version>` in `.vscode/mcp.json`.

## Not in v1

- Autonomous delivery loop (conductor agent, delivery check script). Add it when the six skills
  work well.
- Role agents (Designer, Developer, Reviewer) and hooks.
- QA test plans, Feature-level delivery, wiki publication.
