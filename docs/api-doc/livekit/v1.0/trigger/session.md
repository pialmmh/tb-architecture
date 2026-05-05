# session

Session lifecycle: log in (verify credentials, set cookie), log out (invalidate), introspect (return current user).

## Activated by

- input: [input/api/login](../input/api/login.md)
- input: [input/api/logout](../input/api/logout.md)
- input: [input/api/me](../input/api/me.md)

## What it does

- **login**: BCrypt-verifies the password against the user record. On success, creates a server-side session and emits a `Set-Cookie` header. On failure: 401, no body detail (don't leak whether the username exists).
- **logout**: invalidates the current session if any; no-op otherwise. 200 either way.
- **me**: returns the user behind the current session, or `null` if there is no session.

## Produces

- [output/sessionCookie](../output/sessionCookie.md) — login + logout (clear)
- [output/userView](../output/userView.md) — me + login response body

## Idempotency & retries

Login is not idempotent in the strict sense (re-issues a session each call). Logout is idempotent. `me` is a pure read.

## Notes

Session storage is in-process (Spring `HttpSession`); sticky sessions would be required if we ever ran multiple backend instances behind a load balancer.
