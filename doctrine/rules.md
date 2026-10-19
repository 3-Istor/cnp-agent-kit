# CNP rules

### CNP-01 · Non-root image, one port
**MUST** run as a non-root user and listen on the single port declared in
`deploy/*.yaml`.
Why: the cluster refuses root containers; a port mismatch gives 502s.
Check: Dockerfile scan in CI, smoke check of the release.

### CNP-02 · Health path
**MUST** answer 2xx on the path set in `release.smoke.path` (or
`monitoring.path`) without authentication.
Why: the canary smoke check and the monitoring probe call it.
Check: release smoke step.

### CNP-03 · Configuration from the environment
**MUST** read configuration and secrets from environment variables.
Why: the same image runs everywhere; secrets come from Vault.
Check: review.

### CNP-04 · No secret in the repository
**NEVER** commit a secret, key or `.env` file. `.env.example` lists names only.
Why: every repository is readable by the whole organization and its CI.
Check: `push_files` refuses them, secret scan in CI.

### CNP-05 · Migrations through Alembic
**MUST** change the database schema only through Alembic migrations, never
`create_all` or manual SQL.
Why: the platform backs up, migrates, then starts the version; anything else
skips the backup and the order.
Check: template CI, migration Job.

### CNP-06 · Backward compatible migrations
**MUST** keep a migration compatible with the previous version (expand, then
contract in a later release).
Why: during a canary both versions run against the same database.
Check: squawk lint in CI; `migration:breaking` label as the escape hatch.

### CNP-07 · Releases are tags
**MUST** release by tagging `vX.Y.Z` on main. **NEVER** deploy by changing
the cluster or the image tag by hand.
Why: Git is the source of truth; manual changes are reverted by Argo CD.
Check: Image Updater only follows semver tags.

### CNP-08 · Logs on stdout
**SHOULD** log to stdout, one line per event, JSON when possible.
Why: Loki collects stdout; files in the container are lost.
Check: review.

### CNP-09 · Stateless pods
**MUST NOT** keep state on the pod's disk.
Why: pods move, scale and sleep off-hours; their disk goes with them.
Check: review.

### CNP-10 · Declared resources
**SHOULD** keep requests and limits declared in `deploy/*.yaml`.
Why: the cluster is shared and tight on memory; FinOps reads them.
Check: chart defaults, FinOps recommendations.

### CNP-11 · AGENTS.md
**SHOULD** keep an `ai-context.md` (or `AGENTS.md`) at the root, with the
app's own conventions.
Why: assistants read it before changing the app.
Check: template.

### CNP-12 · No personal data in sandboxes
**NEVER** collect personal or sensitive data in an app of a sandbox or demo
project.
Why: nobody owns its retention or access.
Check: guide cnp-new-app, review.
