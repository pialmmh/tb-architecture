# listInvites()

HTTP API call. `GET /api/meetings/{meetingId}/invites`. Returns every active and revoked magic link for the meeting, newest first.

Authenticated. Host or `ADMIN`. Each row carries `linkType` (`PUBLIC` | `PRIVATE`), `contactChannel` (PRIVATE only), TTL, and any device-fingerprint claims that have already been recorded.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/magicLinkLifecycle](../../trigger/magicLinkLifecycle.md)
