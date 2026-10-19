# Fullstack app (template-app-webapp-python-fastapi-react)

```
backend/app/            FastAPI: main.py, config.py, database.py, models.py
backend/migrations/     Alembic migrations (versions/)
backend/migrate         run by the platform before each new version starts
backend/Dockerfile
frontend/src/           React (Vite): App.jsx, main.jsx
frontend/nginx.conf.template, frontend/Dockerfile
deploy/values-backend.yaml, deploy/values-frontend.yaml
.github/workflows/ci.yml
ai-context.md           the template's own rules for assistants: read it
```

- Backend layers: routers stay thin, logic in services, Pydantic schemas in
  and out, settings from environment variables only.
- **Database changes are Alembic migrations**, never `create_all` and never
  by hand. Add one file in `backend/migrations/versions/`. Make changes
  backward compatible (expand, then contract in a later release): the old
  version keeps running during a canary. The CI lints migrations (squawk).
- The platform takes a database backup, then runs `backend/migrate`, then
  starts the new version. A failed migration stops the release.
- Database credentials and other secrets come from Vault as environment
  variables. New secret → `request_secret`.
- The backend must answer on its health path; the release smoke check calls
  it before any traffic.
- Tests live next to the code (`backend/app/test_*.py`). Add one for each
  new endpoint.
