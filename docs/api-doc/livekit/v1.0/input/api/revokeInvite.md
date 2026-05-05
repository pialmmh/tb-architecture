# revokeInvite()

HTTP API call. `DELETE /api/meetings/{meetingId}/invites/{token}`. Permanently removes a magic link. Already-issued LiveKit tokens are not affected (LK tokens are self-contained JWTs); the magic link can no longer be used to mint new ones.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `token` (path, required) — magic link token (the random string in the share URL).

## Fires trigger

- [trigger/magicLinkLifecycle](../../trigger/magicLinkLifecycle.md)
