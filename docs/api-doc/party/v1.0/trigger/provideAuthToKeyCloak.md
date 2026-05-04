# provideAuthToKeyCloak

Named business action: act as the user-storage backend for Keycloak. Keycloak owns OIDC token issuance; Party owns the user data + password hashes.

## Activated by

- input: [input/api/kcFindUserByUsername](../input/api/kcFindUserByUsername.md) — resolve a user
- input: [input/api/kcValidateCredentials](../input/api/kcValidateCredentials.md) — verify a password

(Later: `kcFindUserById`, `kcSearchUsers` — admin-console paths. Not wired in the minimum.)

## What it does

- Resolves the realm to a tenant id (`realm = "tenant-" + tenant.shortName`).
- For lookup: reads `auth_user` by `(tenantId, email)` and shapes it into a Keycloak SPI user view (id, username, email, firstName, lastName, enabled, roles).
- For validation: BCrypt-verifies the plaintext against `auth_user.password_hash`. Returns `false` for non-`ACTIVE` users even on a correct password.

## Produces

- [output/kcUserView](../output/kcUserView.md) — from lookup
- [output/kcCredentialsValidation](../output/kcCredentialsValidation.md) — from validation

## Notes

- LAN-only — caller must present `X-Party-Kc-Token` matching `party.kc-integration.secret`. No public exposure ever.
- This trigger has **no side-effects on master** (no row writes, no audit log). It is read-only auth federation.
