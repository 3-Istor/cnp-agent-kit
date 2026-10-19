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

- **ask**: the tool answers `needs_user_confirmation`. Ask the user.
- **confirm**: the tool answers `needs_cmp_approval` with a link. A person
  approves in the CMP, then you call again with `confirmation_id`.
- **pull request**: `push_files` opens one; a person reviews and merges.
- While an alert fires on the app, abort and retry are always allowed.
- An owner can open an "autonomous window" (1 to 8 h) on a standard project:
  it then behaves like demo.
- A sandbox holds 3 apps at most.
