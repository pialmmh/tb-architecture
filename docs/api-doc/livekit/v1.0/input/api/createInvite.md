# createInvite()

HTTP API call. `POST /api/meetings/{meetingId}/invites`. Creates a per-recipient `PRIVATE` magic link bound to a specific contact channel (typically email).

Authenticated. Host or `ADMIN`. Rejected for `PUBLIC` meetings (those use one shared link — see [shareInvite](shareInvite.md)). Rejected if the meeting is terminal. Default TTL: 7 days.

## Params

- `meetingId` (path, required) — must be a `PRIVATE` meeting.
- `invitedEmail` (string, required) — contact channel; doubles as the BlockedUser key and (later) the OTP target.
- `invitedName` (string, optional) — display name pre-filled in the join screen.
- `expiresInSeconds` (long, optional, default 604800) — TTL in seconds.

## Fires trigger

- [trigger/magicLinkLifecycle](../../trigger/magicLinkLifecycle.md)
