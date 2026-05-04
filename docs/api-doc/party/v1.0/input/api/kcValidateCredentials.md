# kcValidateCredentials()

HTTP API call. `POST /internal/kc/users/validate-credentials`. Verifies a `(realm, username, password)` triple against the BCrypt hash in `auth_user.password_hash`.

LAN-only. Authenticated by shared secret (`X-Party-Kc-Token`). Constant-time path so unknown-user vs. wrong-password do not leak via timing.

## Params (request body)

- `realm` (string, required) — Keycloak realm; resolved to a tenant.
- `username` (string, required) — typically email.
- `password` (string, required) — plaintext.

## Fires trigger

- [trigger/provideAuthToKeyCloak](../../trigger/provideAuthToKeyCloak.md)
