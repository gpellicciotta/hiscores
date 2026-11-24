<!-- BEGIN plan (managed by plan init) -->
This project tracks its work with plan in `PLAN.md` and `.plan/tasks/`.
Load the `plan-task-usage-guidelines` skill before creating, starting, or changing tasks.
Change `PLAN.md` and task frontmatter only through `plan` commands.
<!-- END plan -->

# Hiscores

A lightweight web application and JSON REST API for tracking top scores across games.
See [index.md](docs/index.md), [requirements.md](docs/requirements.md), and [devops.md](docs/devops.md).

## Commands
- `python scripts/bootstrap-dev-environment.py`: Bootstrap `.venv`, Python tools, Node tools, and build.
- `python scripts/devops.py check`: Run linters, unit tests, and build verification.
- `python scripts/devops.py server start`: Start local MariaDB container and Java server.

## Architecture
- `server/`: Java 25 REST API built on `support.java`, `auth-client-java`, and MariaDB.
- `webapp/`: Vanilla JavaScript SPA built on `support.webapp` and served by `server/`.
- `admin/`: Desktop Admin Console SPA served at `/admin`.
- `docs/`: Requirements, specifications ([data-model-spec.md](docs/specs/data-model-spec.md), [rest-api-spec.md](docs/specs/rest-api-spec.md), [admin-spec.md](docs/specs/admin-spec.md)), and DevOps guide.

## Project Conventions
- Follow `dev-guidelines`, `markdown-guidelines`, and `plan-task-usage-guidelines`.
- Keep all placeholders in `{{kebab-case}}` mustache syntax.

## Boundaries
- Never edit `PLAN.md` or task frontmatter directly; use `plan` CLI commands.
- Never commit `.env` files, credentials, or local secrets.
