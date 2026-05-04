# kcFindUserByUsername()

HTTP API call. `GET /internal/kc/users/by-username`. Resolves an `auth_user` by `(realm, username)` and returns it in Keycloak's user-storage shape.

LAN-only. Authenticated by shared secret (`X-Party-Kc-Token` matched against `party.kc-integration.secret`). Never exposed publicly — Keycloak reaches Party over the WireGuard overlay.

## Params

- `realm` (query, required) — Keycloak realm name. Mapped 1:1 to a Party tenant (`realm = "tenant-" + tenant.shortName`).
- `username` (query, required) — typically the user's email.

## Fires trigger

- [trigger/provideAuthToKeyCloak](../../trigger/provideAuthToKeyCloak.md)
