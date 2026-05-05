# logout()

HTTP API call. `POST /api/auth/logout`. Invalidates the current session.

permitAll — calling it without a session is a no-op.

## Params

(none — session resolved from cookie)

## Fires trigger

- [trigger/session](../../trigger/session.md)
