---
name: cnp-new-app
description: Create a new site or app on CNP from a plain-language request and put it online. Use when someone asks for a new page, site, form, tool or app on the platform.
---

# Create an app on CNP

## 1. Understand the request (one short exchange at most)

Decide alone what you can. Ask only what changes the build, in plain words,
with a recommended answer:

- **Must it store data** (form answers, accounts, history)? No → `static`.
  Yes → `fullstack`. When unsure, start `static`: it is online in minutes and
  can be replaced later.
- **Who should see it?** Default: anyone with a company login. Public access
  is a change of `deploy/` settings and needs the project's agreement (stake
  rules).
- **Which project?** Use the one the user names. A non-developer with no
  project for this gets a sandbox: `create_project` named
  `<username>-sandbox`, stake `sandbox`.

Never collect personal or sensitive data (health, diet, ID numbers) in a
sandbox app. If the request needs it, say so and propose a real project with
an owner.

## 2. Create

1. `create_app(project, name, stack)`. The name becomes the URL and the
   repository: short, lowercase, dashes (`seminar-october`).
2. `wait_for_app(app_id)` until `done` is true (a few minutes; tell the user
   it is being prepared). If the status is `failed`, read `step` and stop:
   explain and offer to retry with another name.

## 3. Write the content

1. `list_app_files` to see the template, then read the files you will change.
2. Follow the template's conventions, see
   [reference/static.md](reference/static.md) or
   [reference/fullstack.md](reference/fullstack.md). Keep the template's
   Dockerfile, CI and `deploy/` as they are.
3. Fullstack: fill the three lines at the top of `AGENTS.md` in the same
   commit: what the app does, the ticket tracker (tool, project key, URL)
   and where its docs live. Ask a developer for the last two in the same
   exchange as step 1; write `none yet` when there is none.
4. `push_files` with every file of the change in one commit, a clear message.

## 4. Put it online

1. `cut_release(app_id)` (first one is `v0.1.0`).
2. Follow with `get_release` until the rollout is finished, or `get_app`
   until the status is `running` for apps without a canary. CI takes a few
   minutes before anything moves: say so.
3. Open nothing yourself: give the user the URL from `get_app`, say it is
   behind the company login, and give the CMP link.

## 5. Offer the next step

One sentence: what you could add next (a section, a map, a form), or how to
share it with the team.
