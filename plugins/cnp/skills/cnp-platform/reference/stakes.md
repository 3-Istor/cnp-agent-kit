# Stake levels and what you may do on your own

The platform decides, server side, from the project's stake level. The source
of truth is `policy.yaml` in the CMP; `get_project_policy` returns the rules of
one project. This page explains them.

| Level | Meaning | Typical example |
|---|---|---|
| sandbox | The user's own experiments | `alice-sandbox` |
| demo | No real users, can break for a while | a manager's test project, a class demo |
| standard | Real internal users (default) | a team tool in daily use |
| critical | Business data, external users, availability commitment | billing, a public site |

| Action | sandbox | demo | standard | critical |
|---|---|---|---|---|
| Read | allow | allow | allow | allow |
| Create a project | ask | ask | ask | ask |
| Create an app | allow | allow | allow | confirm |
| Push application code | allow | allow | allow | pull request |
| Change CI, Dockerfile, `deploy/` | refused | ask | pull request | pull request |
| Release, promote, retry, skip | allow | allow | ask | confirm |
| Abort a rollout | allow | allow | allow | ask |
| Change release settings | allow | allow | ask | confirm |
| Delete an app | ask | ask | CMP only | CMP only |
| Delete a project | ask | ask | CMP only | CMP only |
| Restart an app | allow | allow | ask | confirm |
| Sleep or wake an app | allow | allow | ask | confirm |
| Change deployment settings (`update_app_config`) | allow | allow | ask | confirm |
| Staging: add, remove, wake, schedule | allow | allow | ask | confirm |
| Promotion policy | allow | allow | ask | confirm |
| Run or cancel a test suite | allow | allow | allow | ask |
| Test traffic on a canary | allow | allow | ask | confirm |
| Add or remove a member | ask | ask | ask | confirm |
| Set a budget, apply a cost recommendation | allow | allow | ask | confirm |
| Ignore or forward a cost recommendation | allow | allow | allow | ask |
| Security scan | allow | allow | allow | ask |
| Except a security finding | ask | ask | confirm | confirm |
| Database backup | allow | allow | allow | ask |
| Database restore (not the dry run) | ask | ask | confirm | confirm |
| Analyze a repository to import | allow | allow | allow | ask |
| Apply an import plan | CMP only | CMP only | CMP only | CMP only |
| Link or unlink two apps | confirm | confirm | confirm | confirm |

- **ask**: the tool answers `needs_user_confirmation`. Ask the user.
- **confirm**: the tool answers `needs_cmp_approval` with a link. A person
  approves in the CMP, then you call again with `confirmation_id`.
- **pull request**: `push_files` opens one; a person reviews and merges.
- **CMP only** for an import plan: `import_repo` leaves a draft and a link;
  the user reviews and applies it on that page with their passkey. Nobody
  approves it for you, and calling again never applies it.
- While an alert fires on the app, abort, retry and restart are always
  allowed. Sleep is not: it takes the app down.
- When the CMP asks for a passkey before destructive actions, deleting an
  app or a project, starting a promotion to prod and a full canary promote
  are CMP only at every level: the tool answers `not_allowed_for_assistants`
  with a link for the user.
- An owner can open an "autonomous window" (1 to 8 h) on a standard project:
  it then behaves like demo, except for deleting an app or a project and
  restoring a database, which keep the standard rule.
- A sandbox holds 3 apps at most.

Never done by an assistant, whatever the level: approving an assistant, a
security exception or a promotion that needs an approval, deletion
protection, customer access, closing a month in finance, changing or muting
an alert.
