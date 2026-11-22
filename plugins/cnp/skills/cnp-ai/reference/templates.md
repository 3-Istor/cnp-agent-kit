# Agent templates

The CMP wizard lists them from app-templates `agents/<id>.yaml`. Each file
holds the `agent:` defaults, the recommended models, isolation and daily
budget, the suggested tools with their risk badges, and the risk notes.

| id | For | Trigger | Tools |
|---|---|---|---|
| `assistant` | a general assistant you shape | chat | none until you add some |
| `docs-qa` | answers from the docs, every claim cited | chat | the project's search and read tools |
| `data-analyst` | numbers, tables, charts (code mode) | chat | `run_python` + read-only data tools |
| `app-bot` | reacts to app events: a reader and an actor | event, a2a | reader: `web_search`, `run_python`, public reads; actor: one write tool |
| `ops-diagnosis` | incident diagnosis, actions validated in the CMP | chat or schedule | CMP read tools |
| `pr-review` | pull request review as a comment | event | GitHub read tools + review comment |

Risk badges: `untrusted_content` (reads text written by outsiders),
`private_data`, `external_send` (sends outside, web search included),
`writes`, `executes_code`.

`app-bot` creates `<name>-reader` and `<name>-actor`. The reader reads the
untrusted event, has no private data and no write tool, and answers in a fixed
JSON shape; it calls the actor with that JSON. The actor sees only that JSON
and holds one narrow write tool, scoped by a token the app signs for the
triggering thread. Neither alone can be talked into posting or leaking.

To change a template's behaviour for one agent, edit its `instructions` in
`deploy/values-<agent>.yaml`; keep the closing rules about untrusted data.
