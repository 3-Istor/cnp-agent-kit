# Declaring AI in the app repository

All three blocks go in `deploy/values-<app>.yaml` (or, for an app the CMP has
not linked to a repository yet, `apps/<project>/<app>/values.yaml` in
cnp-projects, written by the CMP). The chart is infra-templates; its
`values.schema.json` is closed, so a typo fails the render in CI.

## App

```yaml
ai:
  enabled: true
```

The app gets:

| Variable | Value |
|---|---|
| `CNP_AI_GATEWAY_URL` | `http://ai-gateway.ai-gateway.svc.cluster.local:8080` |
| `CNP_AI_TOKEN_URL` | Keycloak token endpoint, realm `3istor` |
| `CNP_AI_CLIENT_ID`, `CNP_AI_CLIENT_SECRET` | the app's own client `app-<project>-<app>` (Secret `<app>-ai-oidc`) |

Get a token with `client_credentials`, cache it until a minute before it
expires, and send it as `Authorization: Bearer` to:

- `POST $CNP_AI_GATEWAY_URL/v1/messages`: the Anthropic Messages API, as is;
- `POST $CNP_AI_GATEWAY_URL/a2a/<project>/<agent>`: call an agent (A2A,
  JSON-RPC `message/send` or `message/stream`), if the agent lists
  `app:<app>` in its callers.

Do not set `x-api-key`: the gateway puts the real key. The CMP creates the
client and its Vault secret when a project admin grants the app model access
on the project's AI page.

## MCP

An app that serves MCP (streamable HTTP) on `/mcp`:

```yaml
mcp:
  enabled: true
  port: 8088          # container port; default: the first service port
  tools: [get_thread, search_posts]   # for the catalogue
```

Or an app of type `mcp`, which kmcp runs from the image values (`image`,
`workload.command`, `env` with literal values, `secrets`), port 3000 unless
`mcp.port`.

Agents reach it through the gateway on `/mcp/<project>`, tools named
`<app>__<tool>`. Only the gateway can call it: check the caller's token
anyway (realm `3istor`, claim `project`, and `azp` for a write tool).

## Agent

An app of `type: agent` in the project, values:

```yaml
agent:
  template: app-bot                 # the template it started from
  description: Répond aux mentions @bot
  model: claude-haiku-5-5           # among the agent's rights
  effort: low                       # low | medium | high
  maxSteps: 8                       # model and tool calls per task
  trigger: {type: event}            # chat | event | a2a | schedule
  instructions: |
    ...
  tools: [shop-mcp__get_thread, web_search, run_python]
  output:                           # optional: every answer is this JSON
    name: bot_reply
    schema: {type: object, required: [answer], properties: {answer: {type: string}}}
  calls: [bot-actor]                # other agents it calls
  isolation: kata                   # optional: raise above the rights' minimum
```

Triggers:

- `chat`: the CMP test chat and the CMP API.
- `event`: an app calls it on `/a2a/<project>/<agent>` (callers `app:<app>`).
- `a2a`: another agent calls it (`calls` on the caller, callers
  `agent:<caller>` here).
- `schedule`: `cron: "0 8 * * 1-5"`, `timeZone` (default Europe/Paris) and
  `message`, the task sent at each run.

What the platform renders: a kagent agent whose model and MCP tools go through
a sidecar to the gateway with the agent's own short-lived token, no provider
key, a network policy that lets it reach only the gateway, and the
RuntimeClass of its isolation (gVisor by default). `web_search` is a
model-side tool: the gateway adds it when the rights allow it.
`run_python` runs code in the platform sandbox; inside it, the agent's read
tools are a Python module: `from cnp_tools import tools`.
