# To Postgres

The app reads `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER` and `DB_PASSWORD`.
On an imported app, a variable that held a connection URL is rewritten by
the platform to `postgresql://$(DB_USER):$(DB_PASSWORD)@$(DB_HOST):$(DB_PORT)/$(DB_NAME)`,
so code that reads one URL keeps working.

Use the stack's usual driver and migration tool (SQLAlchemy and Alembic,
Prisma, TypeORM, Flyway...). Create every table and index in a migration.

## From MongoDB

- One collection becomes one table. Fields queried or joined on become
  columns with their type; the rest can stay in a `jsonb` column.
- Embedded arrays that are queried on their own become child tables with a
  foreign key; arrays only read with their parent can stay `jsonb`.
- `_id` ObjectIds: keep them as `text` in a column during the migration so
  links between documents still resolve, then switch to the new keys.
- `$lookup` becomes a `JOIN`; aggregation pipelines become `GROUP BY` and
  window functions. Rewrite them, do not emulate them in application code.
- Indexes: one per query filter; a `jsonb` field filtered often gets a GIN
  or expression index.

## From Neo4j

- Each node label becomes a table, each relationship type a link table
  (`from_id`, `to_id`, plus its properties).
- Fixed-depth traversals become joins; variable-depth ones
  (`-[:FOLLOWS*1..3]->`) become `WITH RECURSIVE` queries with a depth
  limit. Index the link table on both columns.
- If the evidence shows path algorithms (shortest path, PageRank), the
  recommendation is `evaluate`: tell the user, do not rewrite them blindly.

## From Elasticsearch

- Full-text search: a `tsvector` column (generated, with the right
  language configuration) and a GIN index; ranking with `ts_rank`.
- Fuzzy or "contains" matching: the `pg_trgm` extension and a trigram
  index. Check with the user that the extension is acceptable first.
- Aggregations become `GROUP BY`. Relevance tuned with analyzers or scoring
  scripts is a reason for `evaluate`: say it.

## Check before the pull request

- The migration runs from an empty database and the app starts on it.
- Every query of the plan has a test against a real Postgres.
- No `create_all` or manual SQL outside migrations.
