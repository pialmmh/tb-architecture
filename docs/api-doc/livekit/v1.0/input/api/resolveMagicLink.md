# resolveMagicLink()

HTTP API call. `GET /api/magic/{token}`. Resolves a magic link to its meeting + invite metadata. The browser hits this when a user lands on a `/m/{token}` URL — used to render the pre-join screen with the right meeting title, host name, and security mode.

permitAll. Sets a long-lived `meet_dev` HttpOnly device-fingerprint cookie if one isn't already present, used later by [joinMeeting](joinMeeting.md) to bind the link to the device.

## Params

- `token` (path, required) — magic link token.

## Fires trigger

- [trigger/magicLinkClaim](../../trigger/magicLinkClaim.md)
