# shareInvite()

HTTP API call. `POST /api/meetings/{meetingId}/invites/share`. Returns the meeting's public share link, creating one if the meeting doesn't have an active PUBLIC link yet. The returned token is suitable for "Copy link"-style sharing.

Authenticated. Host or `ADMIN`. Rejected for `PRIVATE` meetings (those use per-recipient links — see [createInvite](createInvite.md)). Rejected if the meeting is terminal. Default TTL: 30 days.

## Params

- `meetingId` (path, required) — must be a `PUBLIC` meeting.

## Fires trigger

- [trigger/magicLinkLifecycle](../../trigger/magicLinkLifecycle.md)
