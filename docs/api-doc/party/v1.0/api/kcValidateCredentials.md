# kcValidateCredentials()

HTTP API call. `POST /internal/kc/users/validate-credentials`. Verifies a `(realm, username, password)` triple against the BCrypt hash in `auth_user.password_hash`.

LAN-only. Authenticated by shared secret (`X-Party-Kc-Token`). Constant-time path so unknown-user vs. wrong-password do not leak via timing.

## Params (request body)

- `realm` (string, required) — Keycloak realm; resolved to a tenant.
- `username` (string, required) — typically email.
- `password` (string, required) — plaintext.

## Steps

1. Validate `X-Party-Kc-Token` against `party.kc-integration.secret`.
2. Resolve `realm` to a tenant id.
3. Read `auth_user` by `(tenantId, username)` — if not found, run a dummy BCrypt verify against a sentinel hash to keep the path constant-time.
4. BCrypt-verify `password` against `auth_user.password_hash`.
5. Return `{ valid: true }` only when the password matches **and** `userStatus = ACTIVE`. Suspended / locked / deleted users get `{ valid: false }` even with the correct password.

No master-side writes. `lastLoginAt` is owned by the user-facing login path, not by this SPI.

## Produces

- [output/kcCredentialsValidation](../output/kcCredentialsValidation.md)

## Idempotency & retries

Pure read + crypto compare; safe to retry. No timing leak by construction (constant-time path).

## Response

Universal envelope:

```
{
  success:       bool,
  causeCode:     int,
  causeProtocol: "HTTP",
  description:   string,
  payload:       { valid: bool } | null
}
```

REST, LAN-only SPI. `causeCode` mirrors the HTTP status.

### Success

| causeCode | protocol | description (override)        | payload |
|----------:|----------|-------------------------------|---------|
| 200       | HTTP     | OK                            | [output/kcCredentialsValidation](../output/kcCredentialsValidation.md) — `{ valid: bool }` |

### Failures

| causeCode | protocol | description (override)                                                      | when it fires                                                              |
|----------:|----------|-----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| 400       | HTTP     | request body or query parameters malformed                                  | request body missing `realm` / `username` / `password`, unparseable JSON   |
| 401       | HTTP     | `X-Party-Kc-Token` missing or does not match `party.kc-integration.secret`  | shared-secret check failed                                                 |
| 500       | HTTP     | unhandled error — see server logs (request id in response)                  | DB error, BCrypt verifier crashed                                          |
| 503       | HTTP     | upstream dependency unreachable (Postgres / MariaDB / Keycloak)             | master DB unreachable                                                      |

Every `(causeCode, protocol)` pair above is registered in [../cause-codes.md](../cause-codes.md).

### Notes

- **No `404`.** Unknown user, wrong password, suspended/locked/deleted user — all collapse into `200 OK` with `payload = { valid: false }`. This is deliberate: the lookup runs in constant time so a timing side-channel cannot distinguish "user does not exist" from "user exists, password wrong". See [kcCredentialsValidation](../output/kcCredentialsValidation.md).
- `success: true` with `valid: false` is a normal outcome. Callers that conflate "API success" with "auth success" will get this wrong — check `payload.valid`, not `success`.
- The call has no master-side side effects (no `lastLoginAt` update — that's owned by the user-facing login path), so retries on `500` are safe.
