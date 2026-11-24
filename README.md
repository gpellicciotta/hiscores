# HiNoLuGi Hiscore

A lightweight web application and JSON REST API for tracking top scores across games.

---

## Overview
- Registered users create and manage games and generate sharing tokens.
- Unregistered players can submit scores to games using a sharing token.
- Signed-in users can view all of their own registered scores across games.
- Everyone can browse games and view top-score leaderboards without signing in.
- Administrators can inspect system status, users, games, tokens, and logs in the Admin Console.

---

## Documentation
- [Documentation Index](docs/index.md): Map of requirements, specifications, and operations guides.
- [Requirements](docs/requirements.md): Functional requirements (`R1`-`R9`), technical requirements (`R10`-`R16`), and non-goals.
- [Data Model Specification](docs/specs/data-model-spec.md): MariaDB schema and entity model.
- [REST API Specification](docs/specs/rest-api-spec.md): HTTP endpoints, payloads, and status codes.
- [Admin Console Specification](docs/specs/admin-spec.md): Desktop administration UI routes and screens.
- [DevOps Guide](docs/devops.md): Local setup, testing, versioning, and deployment procedures.

---

## Project Tracking & License
- [PLAN.md](PLAN.md): Task index managed via `plan`.
- [CHANGELOG.md](CHANGELOG.md): Semantic version history.
- [LICENSE.md](LICENSE.md): MIT License.
