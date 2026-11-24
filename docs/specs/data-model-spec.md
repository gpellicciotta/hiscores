# Data Model Specification

MariaDB relational schema, entity definitions, and leaderboard retention rules for Hiscores (`R6`, `R13`).

---

## Database Overview
- **Engine:** MariaDB 11 LTS (and Cloud SQL MySQL/MariaDB compatible SQL).
- **Character Set and Collation:** `utf8mb4` with `utf8mb4_bin` collation.
- **Naming Convention:** Lowercase `kebab-case` names for all tables, columns, and keys.
- **Timestamps:** Stored in UTC; wire payloads format timestamps as `YYYY-MM-DDTHH:MM:SSZ`.

---

## Entity Relationship Diagram
```text
+---------------------+          +---------------------+          +---------------------+
|        users        | 1      * |        games        | 1      * |       scores        |
|---------------------+<---------+---------------------+--------->+---------------------+
| id (PK)             |          | id (PK)             |          | id (PK)             |
| auth-user-id (UK)   |          | owner-user-id (FK)  |          | game-id (FK)        |
| name                |          | name (UK)           |          | player-id (FK, null)|
| email (UK)          |          | description         |          | player-name         |
| realms              |          | hiscore-count       |          | score               |
| timezone            |          | creation-time       |          | achievement-time    |
| creation-time       |          | last-update-time    |          | creation-time       |
| last-activity-time  |          | deletion-time       |          | deletion-time       |
| deletion-time       |          +---------------------+          +---------------------+
+----------+----------+
           | 1
           | *
+----------v----------+
|       tokens        |
|---------------------+
| id (PK)             |
| type                |
| user-id (FK, null)  |
| data                |
| creation-time       |
| last-activity-time  |
| expiry-time         |
+---------------------+
```

---

## Table Definitions

### `versions`
Tracks installed and active schema versions (`version`, `major-version`, `minor-version`, `patch-version`, `label`, `info`, `installation-time`, `deletion-time`).

### `users`
Local shadow profiles synchronized from [Auth](https://auth.pellicciotta.com) (`R1`, `R12`).
New Auth accounts are inserted automatically on their first sign-in callback.

| Column               | Type         | Constraints                                                    | Description                          |
| :------------------- | :----------- | :------------------------------------------------------------- | :----------------------------------- |
| `id`                 | BIGINT       | PRIMARY KEY, AUTO_INCREMENT                                    | Internal user identifier.            |
| `auth-user-id`       | BIGINT       | UNIQUE, NULL                                                   | Subject identifier in Auth.          |
| `name`               | VARCHAR(250) | NOT NULL                                                       | Display name.                        |
| `email`              | VARCHAR(250) | UNIQUE, NOT NULL                                               | Primary email address.               |
| `realms`             | LONGTEXT     | NULL                                                           | JSON array of roles (`["admin"]`).   |
| `timezone`           | VARCHAR(64)  | NOT NULL DEFAULT 'UTC'                                         | Preferred IANA timezone.             |
| `creation-time`      | TIMESTAMP    | NOT NULL DEFAULT CURRENT_TIMESTAMP                             | Creation timestamp.                  |
| `last-activity-time` | TIMESTAMP    | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Last activity timestamp.             |
| `deletion-time`      | TIMESTAMP    | NULL                                                           | Soft-deletion marker (30-day grace). |

### `tokens`
Stores session tokens, sign-in one-time passcodes, and game-scoped `add-score` sharing tokens (`R1`, `R4`).

| Column               | Type      | Constraints                                                    | Description                                        |
| :------------------- | :-------- | :------------------------------------------------------------- | :------------------------------------------------- |
| `id`                 | BIGINT    | PRIMARY KEY, AUTO_INCREMENT                                    | Token identifier.                                  |
| `type`               | ENUM(...) | NOT NULL                                                       | One of `session`, `one-time-code`, `add-score`.    |
| `user-id`            | BIGINT    | NULL, FK `users(id)` ON DELETE CASCADE                         | Owning user (`NULL` for `add-score` tokens).       |
| `data`               | LONGTEXT  | NULL                                                           | Payload (JSON `{"game-id": {{game-id}}}` or code). |
| `creation-time`      | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP                             | Creation timestamp.                                |
| `last-activity-time` | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Last usage timestamp.                              |
| `expiry-time`        | TIMESTAMP | NULL                                                           | Optional expiration timestamp.                     |

### `games`
Registered games (`R2`).

| Column             | Type          | Constraints                                                     | Description                       |
| :----------------- | :------------ | :-------------------------------------------------------------- | :-------------------------------- |
| `id`               | BIGINT        | PRIMARY KEY, AUTO_INCREMENT                                     | Game identifier.                  |
| `owner-user-id`    | BIGINT        | NOT NULL, FK `users(id)` ON DELETE CASCADE                      | Owning user identifier.           |
| `name`             | VARCHAR(100)  | NOT NULL, UNIQUE                                                | Unique game name.                 |
| `description`      | VARCHAR(1000) | NULL                                                            | Human-readable description.       |
| `hiscore-count`    | INT           | NOT NULL DEFAULT 25, CHECK (`hiscore-count` BETWEEN 3 AND 1000) | Top scores retained per game.     |
| `creation-time`    | TIMESTAMP     | NOT NULL DEFAULT CURRENT_TIMESTAMP                              | Creation timestamp.               |
| `last-update-time` | TIMESTAMP     | NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  | Last update timestamp.            |
| `deletion-time`    | TIMESTAMP     | NULL                                                            | Optional soft-deletion timestamp. |

### `scores`
Leaderboard entries per game (`R5`, `R6`, `R7`).

| Column             | Type         | Constraints                                | Description                                  |
| :----------------- | :----------- | :----------------------------------------- | :------------------------------------------- |
| `id`               | BIGINT       | PRIMARY KEY, AUTO_INCREMENT                | Score identifier.                            |
| `game-id`          | BIGINT       | NOT NULL, FK `games(id)` ON DELETE CASCADE | Target game identifier.                      |
| `player-id`        | BIGINT       | NULL, FK `users(id)` ON DELETE SET NULL    | Authenticated submitter, if signed in.       |
| `player-name`      | VARCHAR(250) | NOT NULL                                   | Displayed player name.                       |
| `score`            | BIGINT       | NOT NULL                                   | Numeric score value.                         |
| `achievement-time` | TIMESTAMP(3) | NOT NULL DEFAULT CURRENT_TIMESTAMP(3)      | When the score was achieved.                 |
| `creation-time`    | TIMESTAMP(3) | NOT NULL DEFAULT CURRENT_TIMESTAMP(3)      | When the score record was created on server. |
| `deletion-time`    | TIMESTAMP    | NULL                                       | Optional soft-deletion timestamp.            |

---

## Ordering and Retention Rules
- Leaderboard ordering for a game places the highest numeric score first.
- Ties in numeric score are ordered by earliest `achievement-time` first.
- Remaining ties in both score and `achievement-time` are ordered by earliest `creation-time` first.
- When inserting a new score for a game with `hiscore-count = N`:
  - If fewer than `N` scores exist, or the new score ties or beats the `N`-th score, store it and compute its 1-based rank.
  - Prune any stored scores beyond the top `N` (preserving boundary ties or trimming strictly past `N` deterministically per `R6`).
  - If `N` scores already rank strictly above the new score, do not persist it (`202 Accepted`).
- Changing a game's `hiscore-count` to a smaller value prunes excess lower-ranked scores on update.
