# REST API Specification

HTTP JSON REST API contract between the HiNoLuGi Hiscore server, SPA, and Admin Console.

---

## Conventions and Protocols
- **Base Path:** `/api` on the same origin as the SPA and Admin Console.
- **Content Type:** `application/json; charset=utf-8` for all JSON request and response bodies.
- **Field Naming:** Lowercase `kebab-case` keys across all JSON objects.
- **Timestamps:** UTC ISO-8601 strings (`YYYY-MM-DDTHH:MM:SSZ`); requests accept any ISO-8601 offset, clamped to server receive time when in the future.
- **Authentication:** `Authorization: Bearer {{token}}` using either a user session token or a game-scoped `add-score` token.
- **CORS:** Allow configured origins (from `CORS_ALLOWED_ORIGINS`) with `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, and `Authorization`, `Content-Type` headers.

---

## HTTP Status Codes
- `200 OK`: Request succeeded.
- `201 Created`: Resource created (including accepted leaderboard scores).
- `202 Accepted`: Score ignored because it did not place within the game's top `score-count`.
- `204 No Content`: Deletion succeeded.
- `400 Bad Request`: Invalid identifier, missing mandatory field, or malformed payload.
- `401 Unauthorized`: Missing or invalid credentials.
- `403 Forbidden`: Authenticated user lacks ownership or `admin` role.
- `404 Not Found`: Target user, game, score, or token does not exist.
- `409 Conflict`: Unique constraint conflict (such as duplicate game name).
- `429 Too Many Requests`: Per-IP request throttle exceeded.
- `503 Service Unavailable`: Server is starting or shutting down.

---

## Endpoints

### Authentication and Users (`R1`, `R12`)
| Method   | Path                       | Access              | Description                                                      |
| :------- | :------------------------- | :------------------ | :--------------------------------------------------------------- |
| `GET`    | `/api/auth-token/callback` | Public              | Validate Auth `authorization-code` and return one-time passcode. |
| `POST`   | `/api/auth-token`          | Basic (`email:opc`) | Exchange one-time passcode for a 30-day Bearer session token.    |
| `GET`    | `/api/auth-token`          | Bearer token        | Return metadata for the active session or `add-score` token.     |
| `DELETE` | `/api/auth-token`          | Registered          | Revoke the current Bearer session token.                         |
| `GET`    | `/api/users/me`            | Registered          | Return the authenticated user profile.                           |
| `GET`    | `/api/users`               | Admin               | List all user accounts.                                          |
| `GET`    | `/api/users/{{user-id}}`   | Owner or Admin      | Get user profile (`{{user-id}}` accepts `me`).                   |
| `PUT`    | `/api/users/{{user-id}}`   | Owner or Admin      | Update profile, mark for deletion, or `undo-delete`.             |
| `DELETE` | `/api/users/{{user-id}}`   | Admin               | Permanently delete a user account.                               |

### Games (`R2`, `R3`)
| Method   | Path                     | Access         | Description                                               |
| :------- | :----------------------- | :------------- | :-------------------------------------------------------- |
| `GET`    | `/api/games`             | Public         | List active games; supports optional `?q={{name-query}}`. |
| `POST`   | `/api/games`             | Registered     | Create a game (`201 Created`).                            |
| `GET`    | `/api/games/{{game-id}}` | Public         | Get game metadata.                                        |
| `PUT`    | `/api/games/{{game-id}}` | Owner or Admin | Update `name`, `description`, or `score-count`.           |
| `DELETE` | `/api/games/{{game-id}}` | Owner or Admin | Delete game and cascade its scores and sharing tokens.    |

### Sharing Tokens (`R4`)
| Method   | Path                                                   | Access         | Description                                      |
| :------- | :----------------------------------------------------- | :------------- | :----------------------------------------------- |
| `GET`    | `/api/games/{{game-id}}/add-score-tokens`              | Owner or Admin | List all `add-score` tokens for a game.          |
| `POST`   | `/api/games/{{game-id}}/add-score-tokens`              | Owner or Admin | Create a new `add-score` token (`201 Created`).  |
| `GET`    | `/api/games/{{game-id}}/add-score-tokens/{{token-id}}` | Owner or Admin | Get one `add-score` token for a game.            |
| `DELETE` | `/api/games/{{game-id}}/add-score-tokens/{{token-id}}` | Owner or Admin | Revoke one `add-score` token (`204 No Content`). |
| `DELETE` | `/api/games/{{game-id}}/add-score-tokens`              | Owner or Admin | Revoke all `add-score` tokens for a game.        |

### Scores (`R3`, `R5`, `R6`, `R7`)
| Method   | Path                                         | Access                 | Description                                             |
| :------- | :------------------------------------------- | :--------------------- | :------------------------------------------------------ |
| `GET`    | `/api/games/{{game-id}}/scores`              | Public                 | Get ordered top scores for a game (`?limit={{count}}`). |
| `POST`   | `/api/games/{{game-id}}/scores`              | Owner, Admin, or Token | Submit a score (`201 Created` or `202 Accepted`).       |
| `GET`    | `/api/games/{{game-id}}/scores/{{score-id}}` | Public                 | Get details of a single score.                          |
| `DELETE` | `/api/games/{{game-id}}/scores/{{score-id}}` | Owner or Admin         | Delete a single score (`204 No Content`).               |
| `GET`    | `/api/users/{{user-id}}/scores`              | Self or Admin          | Get personal scores across games (`me` supported).      |

### Operations and Diagnostics (`R9`)
| Method   | Path                       | Access | Description                                               |
| :------- | :------------------------- | :----- | :-------------------------------------------------------- |
| `GET`    | `/api`                     | Public | Return API version and endpoint directory.                |
| `GET`    | `/api/version`             | Public | Return server semantic version.                           |
| `GET`    | `/api/status`              | Public | Return service health, uptime, and database connectivity. |
| `GET`    | `/api/server`              | Admin  | Return detailed server status and active session count.   |
| `DELETE` | `/api/server`              | Admin  | Initiate graceful server shutdown using shutdown token.   |
| `GET`    | `/api/logs`                | Admin  | Return recent in-memory server log entries.               |
| `GET`    | `/api/tokens`              | Admin  | List all active and expired tokens across users/games.    |
| `DELETE` | `/api/tokens/{{token-id}}` | Admin  | Revoke any token by ID.                                   |

---

## Key Payloads

### Create or Update Game
```json
{
  "name": "Snake",
  "description": "Classic arcade snake",
  "score-count": 25
}
```

### Submit Score
- When authenticated as a signed-in user (even when also using a game sharing token via `X-Add-Score-Token` or URL token exchange), the server sets `player-id` to the authenticated user's ID (`R5`).
- When authenticated only with an `add-score` Bearer token, `player-id` is `null` and `player-name` is mandatory.
```json
{
  "player-name": "Hilde",
  "score": 400,
  "achievement-time": "2026-11-24T10:15:00Z"
}
```

### Score Response
```json
{
  "id": 12,
  "game-id": 2,
  "game-name": "Snake",
  "player-id": 3,
  "player-name": "Hilde",
  "score": 400,
  "rank": 1,
  "achievement-time": "2026-11-24T10:15:00Z",
  "creation-time": "2026-11-24T10:15:01Z"
}
```

### Add-Score Token Response
- The `bearer-token` secret is returned on creation (`POST`); list responses omit or mask the raw secret if hashed, or return the shareable bearer code as configured.
```json
{
  "id": 245,
  "game-id": 37,
  "bearer-token": "{{token}}",
  "expiration-time": null
}
```
