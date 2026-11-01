---
name: cnp-change-app
description: Change an existing CNP app (content, feature, fix) and publish the change. Use when someone asks to modify, add to, fix or update an app that already exists on the platform.
---

# Change an app on CNP

1. **Find it.** `whoami`, then `list_apps(project)`; confirm with the user
   when two apps could match. `get_app` gives its stack, URL and whether a
   rollout is in progress. Do not start a change while one is.
2. **Read before writing.** `list_app_files`, then `read_app_file` on every
   file you will touch and on `AGENTS.md` (older apps: `ai-context.md`).
   Follow what is there; do not restructure what you were not asked to.
3. **Change little, in one commit.** `push_files` with the full new content
   of each changed file. Message in the imperative, saying the why when it is
   not obvious. If the answer is a pull request (critical project, or CI and
   deploy files), give the link and stop: a person merges it.
4. **Publish.** `cut_release` with `patch` for a fix or small change,
   `minor` for a new feature, `major` only when users must change something.
   Then follow it (guide `cnp-release`).
5. **Check and report.** Give the URL and what changed, in one or two
   sentences.

## Developers in Claude Code with the repository cloned

Work in the local clone with git as usual, run the tests, push a branch and
open a pull request. Use the CNP tools for what git cannot do: release
(`cut_release`), follow the rollout, logs.
