# Rights (`spec.ai` of the registry record)

Written by the CMP; a project admin changes them in the CMP. Shown here so you
can read them, never edit the record by hand.

```yaml
spec:
  ai:
    enabled: true
    models: [claude-haiku-5-5, claude-sonnet-5-5]
    webSearch: true
    dailyTokenBudget: 2000000          # project: alert only
    apps:   [{name: shop-mcp, enabled: true, dailyTokenBudget: 0}]
    agents:
      - name: bot-reader               # an app of type agent
        owner: jane.doe
        models: [claude-haiku-5-5]
        dailyTokenBudget: 300000       # hard, at the gateway
        tools: [shop-mcp__get_thread, run_python, web_search]
        isolation: gvisor              # runc | gvisor | kata, minimum
        callers: [app:shop-api, user:members]
        calls: [bot-actor]
        status: active                 # suspended = kill switch
```

| Right | Effect |
|---|---|
| `models` | the gateway never serves another model to the agent |
| `webSearch` | without it, `web_search` is stripped from requests |
| `dailyTokenBudget` (agent, app) | `429` once spent, until midnight UTC |
| `tools` | an MCP tool outside the list is unknown to the agent |
| `isolation` | the agent's pods run at least in this runtime |
| `callers` | who may start the agent on `/a2a/<project>/<agent>` |
| `calls` | the agents it may call |
| `status: suspended` | every call refused, at once |

The registry refuses: AI on a critical or off-prem project, an agent app
without rights or rights without an agent app, a model the project does not
allow, `web_search` without `webSearch`, `calls` or `callers` naming nothing
in the project.

Costs: Haiku 5.5 is the default (effort low); Sonnet 5.5 (effort medium)
costs about 20 times more per token. A web search adds about 19,000 input
tokens. The platform's global AI budget stops every call when reached and
alerts at 80 %.
