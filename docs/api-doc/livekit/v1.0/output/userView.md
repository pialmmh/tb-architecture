name: userView
type: json
trigger: [userDirectory](../trigger/userDirectory.md), [session](../trigger/session.md) (me + login response)

Minimal projection of a `User` row. Notable shape:

- `id`, `username`, `email`, `name` — identity
- `role` — `ADMIN` | `AGENT` | `CLIENT`

**Never returned:** `passwordHash` and any internal-only fields. The projection is the only path through which user data leaves the service; raw `User` is not serialised.
