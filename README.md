# CNP Agent Kit

Use the CNP platform from an AI assistant. Ask in plain language: the
assistant creates the app, writes the code, releases it and follows the
rollout, with your company login and your rights.

- **Anyone, nothing to install**: add the connector `https://cmp.3istor.com/mcp`
  in claude.ai or ChatGPT. Step by step: https://cmp.3istor.com/connect-ai
- **Developers in Claude Code**: install this plugin. It configures the
  connection and adds the platform guides and commands.

```
/plugin marketplace add 3-Istor/cnp-agent-kit
/plugin install cnp@cnp-agent-kit
```

Then `/cnp:new <what you want>`, `/cnp:release <app>`, `/cnp:status`,
`/cnp:doctor`.

## What is in here

| Path | What |
|---|---|
| `plugins/cnp/skills/` | The platform guides (Agent Skills). Also served by the CNP MCP server to assistants without the plugin |
| `plugins/cnp/.claude-plugin/plugin.json`, `.mcp.json` | Claude Code plugin |
| `plugins/cnp/plugin.json`, `mcp.json` | Cross-agent plugin manifest (Agent Plugins format, for Codex, Cursor, Copilot) |
| `plugins/cnp/commands/` | Claude Code commands |
| `doctrine/` | The numbered rules an app follows on CNP, with why and how each is checked |
| `evals/briefs/` | Requests to replay after changing a guide or a tool |

## How it works

The CMP serves an MCP server on `/mcp` and is its own OAuth server: the
assistant only talks to the platform, the user signs in with the usual
company login and approves the assistant on a CMP page. Every tool acts with
the user's rights, through the CMP API, and through a policy that scales with
what is at stake in the project (sandbox, demo, standard, critical). Details
in the CMP repository, `backend/app/mcp_server.py` and
`backend/app/services/agents/policy.yaml`.

## Changing a guide

Guides are read by models: short, concrete, one task each, links instead of
copies. The CMP refreshes them from `main` every 10 minutes. Replay the
`evals/briefs/` that touch the guide you changed.
