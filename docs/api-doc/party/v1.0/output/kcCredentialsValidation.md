name: kcCredentialsValidation
type: { valid: bool } (json)
trigger: [provideAuthToKeyCloak](../trigger/provideAuthToKeyCloak.md)

Result of a BCrypt password check, served back to Keycloak. `valid=true` only when the password matches AND the user is `ACTIVE`. Suspended / locked / deleted users always get `valid=false` even with the correct password.

No `lastLoginAt` update happens here — that's owned by the user-facing login path, not by the SPI.
