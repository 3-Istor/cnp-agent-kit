---
name: cnp-run-app
description: "Operate a CNP app or project that already runs: restart or sleep it, change its replicas or resources, manage members, costs and budget, security findings, database backups and restores, read alert rules. Use when someone asks to scale, restart, pause, add a teammate, check or cut costs, deal with a vulnerability, back up or restore data, or asks what alerts watch an app."
---

# Run an app on CNP

Read the current state before any change, say what you will change, then do
it. Every write goes through the project's stake policy (`cnp-platform`).

You act with the user's role on the project. Most changes below need a
project admin (settings, staging, members, backups and restores, cost
recommendations); the budget needs the owner. A tool refuses at once when the
user lacks the role: tell them who can do it instead of asking for an
approval.

## App

| Need | Tools |
|---|---|
| Restart (same version), pause to save money, start again | `app_power` restart, sleep, wake |
| Replicas, CPU and memory, database on or off | `get_app_config`, then `update_app_config` with only the keys to change |

`update_app_config` commits to the app's values file on main; the platform
deploys it in a few minutes. It refuses the keys that have their own tool:
`db` (backups, restores), `release` and `migrations` (release settings),
`image` (releases), `database` and `security` (the CMP). It also refuses
any value the platform manages (the app's identity, its Vault access,
`secrets`, the ingress host...): only replicas, resources, autoscaling,
service, probes, env and config map, pod labels and annotations, monitoring,
log retention, and the ingress switch, its SSO and allowed groups can change. Code
changes go through `cnp-change-app`.

## Project

- Members: `list_members`, `project_member` add (admin, member or guest) or
  remove. Owners are named in the CMP only.
- `delete_project` deletes an empty project: delete its apps first with
  `delete_app`. Sandbox and demo only, after the user confirms. When the CMP
  asks for a passkey it answers `not_allowed_for_assistants`: give the user
  the link, they delete it there. It can answer "deleting": check with
  `whoami`.

## Costs

`get_costs` gives this month so far, the month-end forecast, the budget and
recommendations with their monthly saving. Cloud costs come from the invoice,
about a day late; today is provisional.

- `set_budget`: monthly amount and alert thresholds in percent. Only the
  project's owner may set it.
- `cost_recommendation`: `apply` changes the app as recommended (a commit),
  `ignore` hides it, `notify` sends it to the project's admins. Apply only
  after the user agrees, and say the saving and the risk (a smaller app can
  be slower).

## Security

`get_security` gives the summary, the open findings (most severe first) and
the exceptions. `security_scan` scans an app again after a fix.

`request_security_exception` excepts one finding: `not_affected` with a VEX
justification, `false_positive`, or `accepted_risk` (no justification), with
a statement a reviewer can check and, for an accepted risk, an end date. Some
exceptions apply at once and stop the finding from failing CI, so on
standard and critical projects a person approves the call in the CMP first.
Never except a finding to make a check pass: fix first, except only what is
really not exploitable.

## Data

- `list_backups` shows the app's database and its backups.
- `database_backup` takes one now: do it before anything risky.
- `database_restore` replaces the data. Always run it with `dry_run=true`
  first and tell the user what it would do and what is lost (everything
  written after the backup or the target time). Then, only on a clear yes,
  `dry_run=false`; on standard and critical projects a person approves it in
  the CMP.

## Alerts

`get_alert_rules` lists the alert rules the project or app can have, whether
each is on, and its thresholds. `get_alerts` says what fires now. Changing a
rule, switching it off or muting it is done by the user in the CMP, never by
an assistant: if an alert is noisy, find the cause (`cnp-troubleshoot`) and
tell the user which rule to change.
