name: kcUserView
type: KcUserView (json)
trigger: [provideAuthToKeyCloak](../trigger/provideAuthToKeyCloak.md)

Keycloak-shaped projection of one `auth_user` row, returned to Keycloak's User Storage SPI. Shape:

- `id` — Party `auth_user.id` as string (per SPI contract)
- `username` — typically the email
- `email`, `firstName`, `lastName`
- `enabled` — `true` only if `userStatus == ACTIVE`
- `emailVerified` — `true` (Party assumes verified at create time; revisit when self-service signup ships)
- `tenantId`, `partnerId`
- `roles` — string array of role names attached to the user
