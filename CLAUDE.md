# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

This is a monorepo with two independent sub-projects, each with their own git history, dependencies, and CLAUDE.md:

| Directory | What it is |
|-----------|------------|
| [proxecto-back/](proxecto-back/) | Django 5.2 REST API (Python / Poetry) — see [proxecto-back/CLAUDE.md](proxecto-back/CLAUDE.md) |
| [proxecto-front/](proxecto-front/) | React 19 + Vite SPA (Node / npm) — see [proxecto-front/CLAUDE.md](proxecto-front/CLAUDE.md) |

## Running the full stack

The root [docker-compose.yml](docker-compose.yml) spins up all three services together:

```bash
# Build and start everything (frontend on :80, backend internal :8000, postgres on :5432)
docker compose up -d --build

# Apply migrations after first start (or after model changes)
docker compose exec backend poetry run python manage.py migrate
```

Services:
- **frontend** — Nginx serves the built React SPA on port 80; proxies `/api/` to the `backend` service and serves `/media/` from the shared volume.
- **backend** — Gunicorn running the Django app; exposes port 8000 internally.
- **database** — Postgres 17 with a named volume (`database-data`). Credentials: `postgres/postgres`, DB: `dataWeb`.

The `proxecto-back/media/` directory is bind-mounted into both the backend and frontend containers so uploaded images are served by Nginx in production.

## How the pieces connect

- **Auth:** session-cookie based. Django sets `sessionid` + `csrftoken` cookies; the React frontend reads the CSRF token from the cookie and sends it as a header on mutating requests.
- **API:** all endpoints under `/api/`. In dev the Vite dev server proxies `/api` → `http://localhost:8000`; in production Nginx proxies it to the `backend` container.
- **CORS:** Django allows credentials from the two trusted origins (`http://localhost:5173` and `https://coaching.daw.iesrodeira.com`). Both `SESSION_COOKIE_SECURE` and `CSRF_COOKIE_SECURE` are `True`, so production requires HTTPS.
- **Media:** uploaded images are stored in `proxecto-back/media/talleres/` and served at `/media/` by Nginx in production (or Django itself when `DEBUG=True`).

## Production

- **Domain:** `https://coaching.daw.iesrodeira.com`
- **Stack:** Docker Compose with three services — Nginx (port 80), Gunicorn (internal port 8000), Postgres 17.
- The `backend/` directory at the repo root is empty and unused.
