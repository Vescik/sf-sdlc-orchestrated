# sf-sdlc-orchestrated

A thin GitHub Copilot workspace for Salesforce work on top of the Kantata SX (`KimbleOne__`)
managed package. It takes an ADO work item to a design, an implementation, a deploy to `intDev`,
and a draft PR.

It has a handful of skills and agents, two MCP servers, and no scripts or hooks. The guardrails are the instructions
plus the VS Code, MCP, and GitHub configuration below.

## Contents

| Path | Purpose |
|---|---|
| `.github/copilot-instructions.md` | Routing, tools, org access, confirmations |
| `.github/skills/` | `fetch-ado-item`, `solution-design`, `development`, `continue-development`, `deployment`, `git-workflow`, `design-loop` (run only by `design-loop`), `development-loop` (run only by `dev-loop`) |
| `.github/agents/` | `designer`, `developer`, `reviewer`, `git-agent`, `design-loop`, `dev-loop`, each with only its tools |
| `.github/skills/user-access/` | Domain skill: permission sets and Permission records. Domain skills are listed in `copilot-instructions.md`; new ones start from `docs/domain-skill-template.md` |
| `.github/instructions/` | Package boundaries, Apex rules, Flow rules |
| `.mcp.json` | `ado` (Azure DevOps) and `github` MCP servers, read by the Copilot harness |
| `.vscode/settings.json` | Skills on, read-only commands auto-approved, everything else asks |
| `work-items/` | One folder per work item. See its README |
| `docs/` | Package concept and constraints, filled by the team |
| `force-app/`, `manifest/` | Salesforce DX source |

## Flow

```text
designer   /fetch-ado-item itemId=<ID>    ADO -> work-items/<id>-<slug>/ado-context.md
designer   /solution-design itemId=<ID>   options -> your choice -> design files
reviewer   review the design of <ID>      findings, no edits
developer  implement <ID>                 tasks.md -> force-app/ -> dry run -> your yes -> deploy
reviewer   review the implementation      findings, no edits
git-agent  push and open a draft PR       draft PR through GitHub MCP
```

Or, after you choose an option, let `design-loop` write the design in a loop:

```text
designer     @designer design work item <ID>          options -> stop
design-loop  @design-loop design work item <ID> with option <N>
             designer writes -> structure check -> reviewer applies each domain skill's
             Design check -> fix (max 3 attempts) -> report
you          read the design, set Status: accepted
```

`design-loop` never chooses an option, answers a business question, or accepts the design.
Questions only a human can answer end the loop as `OPEN QUESTIONS`. It logs each step in the chat
and in its report, not in a file.

After design acceptance, let `dev-loop` run development in a loop:

```text
dev-loop   @dev-loop implement work item <ID>
           developer builds -> dry run with RunSpecifiedTests -> fix (max 5 attempts)
           -> reviewer -> dry run with RunLocalTests -> report with the validation job ID
developer  quick deploy of that job ID   your yes -> deploy without rerunning tests
git-agent  push and open a draft PR
```

`dev-loop` never deploys for real. It runs only dry runs, which change nothing in the org, and
logs each step in the chat and in its report, not in a file. Failing tests outside the work item are
reported as blocked, not fixed.

The Copilot harness has no handoff buttons. Each agent ends with the next line to send, such as
`@developer implement the accepted design of work item <ID>`. You check it and send it.

## Setup

1. Install VS Code, the GitHub Copilot extension, and the Salesforce CLI (`sf`). Your Azure DevOps
   organization must sign in with Microsoft Entra ID: the remote ADO MCP server needs it.
2. Log in to the dev org with the alias `intDev`:

   ```bash
   sf org login web --alias intDev --instance-url https://test.salesforce.com
   ```

3. Set the ADO organization name in your shell profile, not in a file in this repo, and restart
   VS Code so it sees the variable:

   ```bash
   export ADO_ORG=<organization>
   ```

4. Open the folder in VS Code. In Copilot Chat, start a new chat, pick **Copilot** in the Session
   Target control before the first prompt, and trust the folder when asked.
5. Run **Chat: Open Customizations**, select **MCP Servers**, and start `ado` and `github`. Sign in
   when asked (**Authenticate**): Microsoft Entra ID for `ado`, GitHub for `github`.
6. Pick the `designer` agent and run `/fetch-ado-item itemId=<ID>`.

The agents use the Copilot schema (`target: github-copilot`). It does not support `handoffs` or
`argument-hint`, so the agents have neither. The Local harness and `.vscode/mcp.json` are not
used.

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
5. **Tools per agent.** Only `designer` has ADO tools, and only `wit_work_item`. Only
   `git-agent` has GitHub tools. `reviewer`, `design-loop`, and `dev-loop` have no editor. The default Copilot agent still has all
   tools, so prefer the named agents.
6. **Narrow MCP scope.** Both servers are remote (HTTP), so there is nothing to install or pin.
   ADO loads only the work item tools (`X-MCP-Toolsets: wit`) in read-only mode
   (`X-MCP-Readonly: true`), so no agent can change ADO. GitHub loads only the `context`, `repos`,
   and `pull_requests` toolsets.

## Not in v1

- A loop over the whole delivery (deploy, push, PR without a human). `design-loop` and `dev-loop`
  cover design and development only.
- Test Strategist agent, hooks.
- QA test plans, Feature-level delivery, wiki publication.
