# Developer Guide

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start status -->
_Up to date._
<!-- sdlc:auto:end status -->

<!-- sdlc:auto:start source -->
Generated from commit `803ed9f` · model `auto`
<!-- sdlc:auto:end source -->

<!-- sdlc:auto:start overview -->
MOVIE Recommendation ("WatchNext") is a single repository holding two generated applications: a Vite + React 19 single-page app in `frontend/` and a FastAPI service in `backend/`. The frontend uses react-router-dom 7, Tailwind CSS 4 (via `@tailwindcss/vite`), TypeScript with an `@/*` path alias to `src/`, and ships a prepared API client (`src/lib/api.ts`) — but the approved screens still render seeded sample data rather than calling the API. The backend is a scaffold: `app/main.py` creates the FastAPI app, enables CORS, creates tables from SQLAlchemy models on startup (no migrations), and exposes exactly one route, `GET /health`; `app/models.py` declares no tables yet and `app/routers/` is empty. Persistence defaults to a local SQLite file (`sqlite:///./app.db`) and can be pointed at the Postgres 16 service in `docker-compose.yml` via `DATABASE_URL`; the compose file also runs a Meilisearch instance, matching the `meilisearch` client in `requirements.txt`.
<!-- sdlc:auto:end overview -->

<!-- sdlc:auto:start prerequisites -->
- Node.js 20+ (Vite 6 / Vitest 2 toolchain)
- npm (bundled with Node.js)
- Python 3.12 (the backend Dockerfile uses python:3.12-slim)
- pip / venv for the backend dependencies
- Docker + Docker Compose (optional — only for the Postgres 16 and Meilisearch services in docker-compose.yml)
<!-- sdlc:auto:end prerequisites -->

<!-- sdlc:auto:start setup -->
1. git clone <repository-url> && cd <repository>
2. cd frontend && npm install
3. npm run dev    # Vite dev server (default http://localhost:5173, an origin the backend already allows via CORS)
4. cd ../backend && python3 -m venv .venv && . .venv/bin/activate
5. pip install -r requirements.txt
6. uvicorn app.main:app --reload    # API on http://localhost:8000, health at /health
7. Optional — start the backing services from the repo root: docker compose up -d db meilisearch
8. Optional — point the API at the compose Postgres instead of SQLite: export DATABASE_URL=postgresql+psycopg://app:app@localhost:5432/app
<!-- sdlc:auto:end setup -->

<!-- sdlc:auto:start run -->
| Name | Command | Description |
|---|---|---|
| frontend dev | `cd frontend && npm run dev` | Runs the Vite dev server for the React SPA. |
| frontend build | `cd frontend && npm run build` | Bundles the app with Vite. Note: it does not typecheck. |
| frontend preview | `cd frontend && npm run preview` | Serves the production build locally (Playwright uses port 4173). |
| typecheck | `cd frontend && npm run typecheck` | Runs `tsc --noEmit` over `src/`. |
| lint | `cd frontend && npm run lint` | ESLint over the frontend; `npm run lint:fix` applies fixable fixes. |
| format | `cd frontend && npm run format` | Prettier write; `npm run format:check` verifies without writing. |
| backend dev | `cd backend && uvicorn app.main:app --reload` | Runs the FastAPI service with autoreload on port 8000. |
| backend lint | `cd backend && ruff check .` | Ruff lint over the backend; `ruff format .` formats it. |
| services | `docker compose up -d` | Starts PostgreSQL 16 (port 5432, app/app/app) and Meilisearch v1.10.3 (port 7700). |
| backend image | `cd backend && docker build -t movie-recommendation-api . && docker run -p 8000:8000 movie-recommendation-api` | Builds and runs the backend container (uvicorn on 0.0.0.0:8000, non-root user, /health healthcheck). |
<!-- sdlc:auto:end run -->

<!-- sdlc:auto:start structure -->
| Path | Purpose |
|---|---|
| `.devcontainer/` | Dev container definition for editors that support it. |
| `.quorq/` | Platform metadata (handoff.json) written by the generator. |
| `backend/` | FastAPI service: requirements.txt, Dockerfile, pytest.ini, ruff.toml. |
| `backend/app/` | Application package: main.py (app, CORS, lifespan, /health), database.py (engine/session/get_db), models.py, schemas.py, services.py. |
| `backend/app/routers/` | Generated routers, one per architecture component that owns endpoints — currently empty. |
| `backend/tests/` | pytest suite with conftest.py; split into unit/ and integration/ (integration/test_health.py covers GET /health). |
| `frontend/` | Vite + React SPA: package.json, vite.config.ts, tsconfig.json, eslint.config.js, playwright.config.ts, index.html. |
| `frontend/src/` | Application source: main.tsx entry (BrowserRouter with Vite BASE_URL), index.css, test-setup.ts. |
| `frontend/src/lib/` | Shared vocabulary: api.ts (fetch wrapper, base URL, session token), ui.tsx, icons.tsx, navigate.ts. |
| `frontend/e2e/` | Playwright acceptance specs plus support helpers (e2e/support/admin.ts). |
<!-- sdlc:auto:end structure -->

<!-- sdlc:auto:start conventions -->
- Two applications in one repository: `frontend/` (SPA) and `backend/` (API), each with its own dependency manifest and tooling config.
- Backend layering: `main.py` wires the app, `database.py` owns the engine/session and the request-scoped `get_db` dependency, `models.py` holds SQLAlchemy models on a shared `DeclarativeBase`, `schemas.py` holds Pydantic request/response models, routers live one-per-component under `app/routers/`.
- Configuration comes from environment variables with working local defaults: `DATABASE_URL` (defaults to `sqlite:///./app.db`), `ALLOWED_ORIGINS` (defaults to the Vite dev origins), `VITE_API_BASE_URL` (defaults to http://localhost:8000), `VITE_API_SECRET_KEY`.
- Unimplemented endpoints return the typed `StubResponse` placeholder rather than raising, so the OpenAPI document stays complete.
- Schema is created from models at startup via the FastAPI lifespan hook; there are no migrations yet.
- Frontend imports use the `@/` alias for `src/` (configured in both vite.config.ts and tsconfig.json).
- Vitest component tests live beside the components they cover under `src/**/*.{test,spec}.{ts,tsx}`; Playwright specs stay in `e2e/` so the two runners don't collide.
- All HTTP from the frontend goes through `apiFetch` in `src/lib/api.ts`, which attaches a bearer token (session token from localStorage, otherwise the shared secret).
- Python is formatted and linted with Ruff; TypeScript with ESLint + Prettier (eslint-config-prettier in use).
- TypeScript is `strict` but with `noImplicitAny` and `strictNullChecks` disabled.
<!-- sdlc:auto:end conventions -->

<!-- sdlc:auto:start testing -->
| Name | Command | Description |
|---|---|---|
| frontend unit | `cd frontend && npm test` | Vitest component tests in jsdom (`vitest run --passWithNoTests`) using src/test-setup.ts. |
| frontend coverage | `cd frontend && npx vitest run --coverage` | Runs the same tests with the v8 coverage provider, writing text and json-summary reports to ./coverage. |
| e2e install | `cd frontend && npm run test:e2e:install` | Downloads the Chromium browser Playwright needs. |
| e2e | `cd frontend && npm run test:e2e` | Playwright specs from e2e/. Without E2E_BASE_URL it boots the backend (from backend/.venv) and a built preview on port 4173; with E2E_BASE_URL set it runs against that URL. |
| backend tests | `cd backend && pytest` | Runs the pytest suite under tests/ (pythonpath set to backend/ so `import app` resolves). |
| backend coverage | `cd backend && pytest --cov=app` | Runs the suite with pytest-cov coverage over the app package. |
<!-- sdlc:auto:end testing -->

<!-- sdlc:auto:start troubleshooting -->
| Problem | Solution |
|---|---|
| `pytest` fails with `ModuleNotFoundError: No module named 'app'`. | Run pytest from the `backend/` directory — pytest.ini sets `pythonpath = .` relative to that folder. |
| Browser calls from the SPA are blocked by CORS. | The backend allows http://localhost:5173 and http://127.0.0.1:5173 by default. For any other origin, set ALLOWED_ORIGINS to a comma-separated list before starting uvicorn. |
| The backend cannot connect to a database. | It defaults to SQLite (./app.db) and needs no server. If you set DATABASE_URL to Postgres, start it first with `docker compose up -d db` (user/password/db all `app`, port 5432). |
| `npm run test:e2e` fails to start the API with a missing interpreter. | playwright.config.ts launches `../backend/.venv/bin/python`. Create it first: `cd backend && python3 -m venv .venv && .venv/bin/pip install -r requirements.txt`. Alternatively set E2E_BASE_URL to a running deployment to skip the local servers. |
| `npm run build` succeeds but deployment fails on type errors. | `vite build` does not typecheck. Run `npm run typecheck` (tsc --noEmit) and fix the reported errors — commonly untyped `useState({})` later read for named fields. |
| The frontend shows sample data instead of live API results. | Expected: the generated screens run on seeded sample data. Wire a screen to the API through `apiFetch` in src/lib/api.ts and set VITE_API_BASE_URL if the API is not on http://localhost:8000. |
| Playwright Chromium crashes in a container. | The config already passes --disable-dev-shm-usage; if it still crashes, increase the container's /dev/shm size or run the suite against a deployed URL via E2E_BASE_URL. |
<!-- sdlc:auto:end troubleshooting -->
