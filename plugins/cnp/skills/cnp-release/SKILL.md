---
name: cnp-release
description: Release a version of a CNP app and follow or act on its rollout (canary steps, promote, retry, skip, abort). Use when someone asks to release, publish, deploy a version, or asks what a rollout is doing.
---

# Releases on CNP

## How a release works

`cut_release` tags `vX.Y.Z` on main. CI builds the image (a few minutes),
the platform picks the new version up and deploys it:

- **rolling** apps switch to the new version once it is healthy;
- **canary** apps first run a smoke check on the new version, then send it a
  growing share of traffic, measuring its error rate at each step. Some steps
  wait for a decision (promote). Users stay on the version that served them
  first.

The platform fixes forward: a bad version is replaced by a new release, not
rolled back by hand. During a rollout, `abort` sends all traffic back to the
stable version.

## Following one

`get_release` shows the steps, the traffic split, the measured values and
`possible_actions`. Poll it every minute or so, and tell the user in one line
when something changes. `list_releases` gives the history.

## Acting on one

`release_action` with:

- `promote`: release the step waiting for a decision (`full=true` goes to
  100 %). Only when the checks are green, or the user explicitly decides.
- `retry`: run a failed or inconclusive check again. Inconclusive usually
  means not enough traffic.
- `skip`: skip a check. Needs a reason, and the user's decision: say what the
  check protects against.
- `abort`: stop and send traffic back to stable. Prefer it to waiting when
  errors rise.

A canary on an app with no real users stays inconclusive: `release_traffic`
sends test requests (up to 20 per second for 15 minutes) so the checks have
something to measure. Stop it with `action="stop"`.

How the app is released (strategy, error rate threshold, smoke path):
read `get_release`, then `update_release_settings` with only the fields to
change. Refused while a canary runs.

The stake policy may answer `needs_user_confirmation` or `needs_cmp_approval`;
see `cnp-platform`.

## Staging and promotion

An app with a staging environment gets release candidates `vX.Y.Z-rc.N`, which
run in staging first. `get_environments` shows both versions, the promotion
policy (test suites, soak time, approval) and past promotions.

- `promotion(action="start")` promotes what runs in staging to prod. With
  `emergency=true` it skips the soak time and needs a reason; use it only
  when the user asks. `rollback_to` promotes an earlier revision.
- A step waiting for a decision: `promotion(action="accept_step")`. A
  promotion that needs an approval is approved by a person in the CMP.
- `promotion(action="cancel")` stops one in progress.
- `ci_run` runs a suite of the policy on an environment; `staging` adds,
  removes, wakes or schedules the staging environment. Hours are cron
  expressions in the staging's timezone, e.g. `0 19 * * 1-5` to sleep at
  19:00 on weekdays; what you do not give is kept.

## Database migrations

On fullstack apps the platform backs up the database, then runs the
migrations, before the new version starts. If the migration fails, the
release stops and nothing changed. Fix the migration and release again.
