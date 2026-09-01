# Local Setup

Use this guide to run Camille on a local machine with seeded demo data.

## Prerequisites

- Git with submodule support
- Docker and Docker Compose
- `just`
- Node.js `22.x` and PNPM `9.x` for the API
- Node.js `20.19.0` and Bun `1.3.4` for the web workspace

## Clone And Submodules

```bash
git clone --branch production --recurse-submodules git@github.com:dauphaihau-camille/camille.git
cd camille
git submodule update --init --recursive
```

## Local URLs

| App | URL |
| --- | --- |
| API | `http://localhost:5100` |
| API Docs | `http://localhost:5100/docs` |
| App | `http://localhost:5102` |
| Marketing | `http://localhost:5101` |
| Grafana | `http://localhost:23001` |
| Bull Board | `http://localhost:5100/ops/queues` |

## API Setup

Run API commands from `apps/api`.

```bash
cd apps/api
just infra-up
just api-install
cp api/.env.example api/.env
just db-migration-up
just seed-full
```

Set `CORS_ALLOWED_ORIGINS=http://localhost:5102,http://localhost:5101` in `api/.env` when testing the local app and marketing site from the browser.

`just seed-full` clears the database and seeds the committed demo users. For richer document scenarios, use `just db-fresh-realistic`. For large navigation and pagination checks, use `just db-fresh-huge`.

## Run API

Start the API and worker in separate terminals.

```bash
cd apps/api
just api-up-observability
```

```bash
cd apps/api
just api-worker-up-observability
```

Useful local endpoints after startup:

- API docs: `http://localhost:5100/docs`
- OpenAPI JSON: `http://localhost:5100/docs/openapi.json`
- Health: `http://localhost:5100/health`
- Readiness: `http://localhost:5100/health/ready`
- Queues: `http://localhost:5100/ops/queues`
- Grafana: `http://localhost:23001`

## Web Setup

Run web commands from `apps/web`.

```bash
cd apps/web
bun install
```

Copy app environment examples when local API access is needed.

```bash
cp apps/app/.env.example apps/app/.env.local
cp apps/marketing/.env.example apps/marketing/.env.local
```

Confirm these values point to the local API:

```bash
NEXT_PUBLIC_API_BASE_URL=http://localhost:5100
NEXT_PUBLIC_API_VERSION=v1
```

## Run Web Apps

Start the product app:

```bash
cd apps/web
bun run dev:app
```

Open `http://localhost:5102`.

Start the marketing site in a separate terminal:

```bash
cd apps/web
bun run dev:marketing
```

Open `http://localhost:5101`.

## Seeded Local Accounts

The demo seed includes these accounts in `apps/api/seed-data/auth-users.tsv`:

| Role | Email | Password |
| --- | --- | --- |
| Admin | `admin@example.com` | `password123` |
| Member | `member@example.com` | `password123` |

Optional local-only accounts can be added in `apps/api/seed-data/auth-users.local.tsv`. Start from `apps/api/seed-data/auth-users.local.example.tsv`.

## Quick Run Check

- API readiness endpoint returns healthy.
- App login, logout, workspace entry, document editing, search, favorites, trash, and settings routes load.
- Marketing homepage and public share route handling load.
- Auth state survives refresh and redirects protected routes correctly.
- Seeded realistic data appears when using `just db-fresh-realistic`.
- Queue worker connects to Redis and processes expected background jobs.
- Observability stack receives local logs, metrics, and traces when using observability commands.

## Common Issues

- Missing API env: copy `apps/api/api/.env.example` to `apps/api/api/.env`.
- Empty workspace or missing scenario data: run `just db-fresh-realistic` from `apps/api`.
- Very large local data set: use `just db-fresh-demo` to return to the small demo seed.
- Port conflicts: stop the existing process or change the app port in the app-specific command or environment.
- Worker queue errors: confirm Redis is running with `just infra-up` from `apps/api`.
- Web API request errors: confirm `NEXT_PUBLIC_API_BASE_URL` points to `http://localhost:5100`.
