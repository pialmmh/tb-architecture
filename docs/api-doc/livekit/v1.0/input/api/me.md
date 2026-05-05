# me()

HTTP API call. `GET /api/auth/me`. Returns the user record bound to the current session, or `null` if no session.

permitAll. The UI uses this on boot to decide whether to show the login screen or the meetings dashboard.

## Params

(none — session resolved from cookie)

## Fires trigger

- [trigger/session](../../trigger/session.md)
