# Requirements

Functional requirements, technical constraints, and non-goals for Hiscores.
Each requirement has a stable identifier (`R1`, `R2`, ...) for specifications and tasks to cite.

---

## Functional Requirements

### Authentication and User Accounts
- R1: Rely on [Auth](https://auth.pellicciotta.com) for user registration, sign-in, and identity.
  Any new Auth account can sign in on its first visit without prior invitation.
  Users can inspect their profile, soft-delete their local account with a 30-day recovery period, and undo deletion.

### Games and Public Leaderboards
- R2: Registered users can create, update, and delete games they own.
  Each game has a unique name, an optional description, and a configurable maximum leaderboard size.
- R3: Any visitor can list all active games, view game details, and read top scores per game without signing in.

### Sharing Tokens and Score Submission
- R4: Game owners and administrators can create, list, and revoke game-scoped sharing tokens (`add-score` tokens).
  Unregistered users and external clients holding a valid token can submit scores for that specific game.
- R5: Game owners, administrators, and holders of a valid `add-score` token can submit scores to a game.
  When a signed-in user submits a score (including via a sharing token), attach their `player-id` automatically.

### Score Ordering, Retention, and Personal Scores
- R6: Rank scores per game first by highest score, then (when equal) oldest achievement time first.
  If both score and achievement time are equal, sort by oldest score creation time first.
  Keep at most the game's configured top-score count (default `25`, range `3` to `1000`), ignoring lower scores on write.
- R7: Any authenticated user can query their own registered scores across all games.
  Personal scores are grouped alphabetically by game name and ordered within each game from highest rank to lowest.

### Web User Interfaces
- R8: Provide a responsive Single-Page Application (SPA) at `/` for all public and player workflows.
  Include Dashboard, Game Leaderboard, Token Score Submission (`#/games/{{game-id}}?token={{token}}`), Game Create/Edit, Sharing Token Manager, My Games, My Scores, and Account views.
- R9: Provide a desktop Admin Console web UI at `/admin` restricted to `admin` users.
  Include Overview (server health, version, active sessions), Users, Games and Tokens, and recent Server Logs screens.

---

## Technical Requirements
- R10: Comply with [dev-guidelines](https://github.com/gpellicciotta/dev-guidelines) for repository layout, `kebab-case` naming, US English, Markdown style, commits, and semantic versioning.
- R11: Build the REST API and static web server in Java 25 using [support.java](https://github.com/gpellicciotta/support.java), Gradle, and Docker.
- R12: Delegate authentication to [Auth](https://auth.pellicciotta.com) using `auth-client-java` for token validation and [todos.webapp](https://github.com/gpellicciotta/todos.webapp) auth patterns.
- R13: Use MariaDB as the relational database with `utf8mb4_bin` collation and `kebab-case` identifiers.
- R14: Build the SPA and Admin Console in vanilla ES modules on [support.webapp](https://github.com/gpellicciotta/support.webapp) without a JavaScript build step.
- R15: Build development, test, database, and deployment automation scripts on [support.python](https://github.com/gpellicciotta/support.python).
- R16: Plan and track all implementation work with [plan](https://github.com/gpellicciotta/plan).

---

## Non-Goals
- Providing standalone client libraries.
- Doing own authentication.
- Enabling offline use.

---

## Specifications
The technical specifications translate these requirements into concrete designs:
- [data-model-spec.md](specs/data-model-spec.md): Relational MariaDB schema, tables, and ordering semantics.
- [rest-api-spec.md](specs/rest-api-spec.md): HTTP REST endpoints, authorization rules, and payloads.
- [admin-spec.md](specs/admin-spec.md): Admin Console layout, routes, and screens.
