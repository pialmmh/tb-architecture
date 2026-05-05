# kcFindUserByUsername()

HTTP API call. `GET /internal/kc/users/by-username`. Resolves an `auth_user` by `(realm, username)` and returns it in Keycloak's user-storage shape.

LAN-only. Authenticated by shared secret (`X-Party-Kc-Token` matched against `party.kc-integration.secret`). Never exposed publicly — Keycloak reaches Party over the WireGuard overlay.

## Params

- `realm` (query, required) — Keycloak realm name. Mapped 1:1 to a Party tenant (`realm = "tenant-" + tenant.shortName`).
- `username` (query, required) — typically the user's email.

## Steps

1. Validate `X-Party-Kc-Token` against `party.kc-integration.secret`.
2. Resolve `realm` to a tenant id (`realm = "tenant-" + tenant.shortName`).
3. Read `auth_user` by `(tenantId, email)` from the master DB.
4. Shape the row into the Keycloak SPI user view (id as string, username, email, firstName/lastName, enabled, roles).

Read-only auth federation — no master-side writes, no audit row.

## Produces

- [output/kcUserView](../output/kcUserView.md)

## Idempotency & retries

Pure read; safe to retry without coordination.

## Response

Universal envelope:

```
{
  success:       bool,
  causeCode:     int,
  causeProtocol: "HTTP",
  description:   string,
  payload:       KcUserView | null
}
```

REST, LAN-only SPI. `causeCode` mirrors the HTTP status.

### Success

| causeCode | protocol | description (override)        | payload |
|----------:|----------|-------------------------------|---------|
| 200       | HTTP     | OK                            | [output/kcUserView](../output/kcUserView.md) |

### Failures

| causeCode | protocol | description (override)                                                      | when it fires                                                              |
|----------:|----------|-----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| 400       | HTTP     | request body or query parameters malformed                                  | `realm` or `username` query param missing                                  |
| 401       | HTTP     | `X-Party-Kc-Token` missing or does not match `party.kc-integration.secret`  | shared-secret check failed — never expose this endpoint to the public net  |
| 404       | HTTP     | Not Found                                                                   | no `auth_user` row for `(tenantId resolved from realm, username)`, or the realm does not resolve to an active tenant |
| 500       | HTTP     | unhandled error — see server logs (request id in response)                  | DB error, unexpected exception                                             |
| 503       | HTTP     | upstream dependency unreachable (Postgres / MariaDB / Keycloak)             | master DB unreachable                                                      |

Every `(causeCode, protocol)` pair above is registered in [../cause-codes.md](../cause-codes.md).

### Notes

- `401` is the only auth-related failure. Realm-not-found and user-not-found both collapse to `404`. This is intentional — Keycloak treats both as "user does not exist here, try the next provider".
- The call has no master-side side effects, so retries on `500` are always safe.
