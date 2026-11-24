# Admin Console Specification

Desktop web administration interface served at `/admin` (`R9`, `R14`).

---

## Scope and Layout
- Served from `admin/` at `/admin` on the same origin as the REST API, requiring no CORS.
- Built with vanilla ES modules using the vendored [support.webapp](https://github.com/gpellicciotta/support.webapp) design system at `/vendor/design/` and `neutral` look.
- Uses the wide shell layout: sidebar, top bar with account menu, main content area, and optional detail side panel.
- Restricted to authenticated users holding the `admin` realm; non-admins see an access-denied notice with a link back to `/`.
- Uses hash routes `#/overview`, `#/users`, `#/users/{{user-id}}`, `#/games`, `#/tokens`, and `#/logs`, defaulting to `#/overview`.

---

## Screens

### Overview (`#/overview`)
- Calls `GET /api/server` and polls every 30 seconds while visible.
- Displays server status, uptime, version, environment, database status and version, and active session count.

### Users (`#/users`, `#/users/{{user-id}}`)
- Calls `GET /api/users` and renders a filterable table of ID, name, email, roles, last active time, and account state.
- Selecting a row opens the side detail panel with profile edit (`PUT /api/users/{{user-id}}`), soft-delete/restore, and permanent delete (`DELETE /api/users/{{user-id}}`).
- Prevents admins from deleting or marking their own account.

### Games and Tokens (`#/games`, `#/tokens`)
- `#/games` lists all games across owners (`GET /api/games`) with owner info, `score-count`, and admin delete/token-inspection actions.
- `#/tokens` calls `GET /api/tokens` to show active and expired session and `add-score` tokens, with in-place revocation (`DELETE /api/tokens/{{token-id}}`).
- Marks the caller's own active session token as `Your session` and disables its Revoke button.

### Logs (`#/logs`)
- Calls `GET /api/logs` and displays recent in-memory server log entries (time, severity chip, logger name, message) with level and text filtering.
