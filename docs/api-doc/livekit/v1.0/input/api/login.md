# login()

HTTP API call. `POST /api/auth/login`. Authenticates a registered user and establishes a session.

permitAll on the route — the call itself is the auth boundary. On success the response sets a session cookie that subsequent authenticated routes look for.

## Params

- `username` (string, required) — username or email.
- `password` (string, required) — plaintext, verified server-side against the hashed credential.

## Fires trigger

- [trigger/session](../../trigger/session.md)
