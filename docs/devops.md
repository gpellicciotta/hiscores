# DevOps Guide

Development environment, versioning, testing, and deployment operations for HiNoLuGi Hiscore (`R10`, `R11`, `R13`, `R15`).

---

## Development Environment

### Prerequisites
- Java 25 or later.
- Python 3.11 or later.
- Node.js 22.13 or later with `npm`.
- Docker Engine (for local MariaDB 11 container and container builds).
- The [plan](https://github.com/gpellicciotta/plan) CLI for task tracking.

### Setup
Bootstrap a fresh checkout once:

```bash
python scripts/bootstrap-dev-environment.py
```

Run checks, fast tests, or the local server via `scripts/devops.py` (built on [support.python](https://github.com/gpellicciotta/support.python)):

```bash
python scripts/devops.py check
python scripts/devops.py test
python scripts/devops.py db start
python scripts/devops.py server start
```

---

## Versioning and Releasing

### Updating Versions
- [CHANGELOG.md](../CHANGELOG.md) and Gradle/package manifests track the project semantic version in lockstep, starting from `v0.1.0-pre`.
- Let `plan tasks stop {{task-id}} --status completed` increment the patch version automatically; never edit patch numbers by hand.

### Making a Release
Preview and cut a release using `scripts/devops.py`:

```bash
python scripts/devops.py release --dry-run
python scripts/devops.py release --yes
```

---

## Test Environments
- Local unit and contract tests run against an in-memory repository and an ephemeral `mariadb:11` container.
- End-to-end browser tests run with Playwright against the running server and mocked or local [Auth](https://auth.pellicciotta.com) endpoints.

---

## Production Environments
- **Compute:** Google Cloud Run container running the Java 25 server and serving `webapp/` and `admin/`.
- **Database:** Managed Cloud SQL MariaDB/MySQL instance (`hiscore` schema).
- **Identity:** Relying-party registration `hiscore` on [Auth](https://auth.pellicciotta.com) with secret stored in GCP Secret Manager.

---

## Pushing to Production
Deploy a tagged release with `scripts/devops.py deploy` after explicit operator approval:

```bash
python scripts/devops.py deploy v{{version}} --dry-run
python scripts/devops.py deploy v{{version}} --yes
```
