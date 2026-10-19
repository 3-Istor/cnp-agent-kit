# Words users may use

| They say | They mean | On CNP |
|---|---|---|
| "my space", "my team's area" | a project | `whoami` lists them |
| "a site", "a page", "a mini-site" | a static app | `create_app` stack `static` |
| "an app that saves things", "a form that keeps answers" | a fullstack app | `create_app` stack `fullstack` |
| "put it online", "publish", "deploy" | a release | `push_files` then `cut_release` |
| "go back", "undo", "it was better before" | stop the rollout, or release a fix | `release_action abort` during a rollout; otherwise fix forward with a new release |
| "who can see it" | SSO protection | apps are behind the company login by default |
| "the link" | the app URL | `get_app` → `url` |
| "it's broken", "it's slow" | troubleshooting | guide `cnp-troubleshoot` |
| "a password", "an API key" | a secret | `request_secret`, never in files |
