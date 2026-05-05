# Party — Cause Codes (v1.0)

The complete cause-code registry for Party at version 1.0. Every Response row in this version references a row here.

## Reading the table

- **value** — wire value (int or string).
- **protocol** — `HTTP` (REST status code) | `PARTY` (service-custom) | `SIP` / `SS7` / `GRPC` / … (pass-through from upstream).
- **description** — standard / original meaning. For `HTTP`, the IANA reason phrase.
- **override_description** — what Party puts in the response envelope's `description` field. `—` means we send `description` verbatim.

All Party APIs in v1.0 are REST and emit HTTP status codes. Custom `PARTY` codes are reserved for future gRPC/internal-Java use and start at 1000.

## Codes

| value | protocol | description           | override_description                                            |
|------:|----------|-----------------------|-----------------------------------------------------------------|
|   200 | HTTP     | OK                    | —                                                               |
|   201 | HTTP     | Created               | tenant provisioned                                              |
|   400 | HTTP     | Bad Request           | request body or query parameters malformed                      |
|   401 | HTTP     | Unauthorized          | `X-Party-Kc-Token` missing or does not match `party.kc-integration.secret` |
|   404 | HTTP     | Not Found             | —                                                               |
|   409 | HTTP     | Conflict              | shortName already taken — pick another                          |
|   422 | HTTP     | Unprocessable Entity  | validation failed — see `payload.errors[]`                      |
|   500 | HTTP     | Internal Server Error | unhandled error — see server logs (request id in response)      |
|   503 | HTTP     | Service Unavailable   | upstream dependency unreachable (Postgres / MariaDB / Keycloak) |

## Notes

- `kcValidateCredentials` deliberately returns `200 OK` with `payload = {valid: false}` for unknown users and wrong passwords alike — constant-time path, no timing leak. There is no `404` row for that API.
- `503` is emitted whenever the trigger detects a downed dependency *before* state has been mutated. Once a `tenantProvisioning` run has written the master row in `PROVISIONING`, partial-failure is `500` with the request id; the next `createTenant()` call resumes from the failure point (per trigger idempotency).
- No `PARTY` custom rows yet. Reserved for future gRPC / internal Java APIs that need finer granularity than HTTP gives. Numeric values start at `1000`.
