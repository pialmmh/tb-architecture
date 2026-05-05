# emitClientEvent()

HTTP API call. `POST /api/meetings/{meetingId}/events/client`. Lets the browser append events the server doesn't observe directly: hand-raises, screen-share starts, file shares, mentions, poll lifecycle. The server validates `type` against a strict whitelist and rejects anything else (400).

permitAll — guests join meetings without a session, so the actor identity is taken from the body, not derived. The whitelist exists precisely because anyone can call this.

## Params

- `meetingId` (path, required)
- `type` (string, required) — must be one of: `HAND_RAISED`, `HAND_LOWERED`, `SCREEN_SHARE_STARTED`, `SCREEN_SHARE_STOPPED`, `PARTICIPANT_LEFT`, `FILE_SHARED`, `MENTION`, `POLL_CREATED`, `POLL_CLOSED`.
- `actorIdentity` (string, required) — LiveKit identity of the emitter.
- `actorName` (string, optional) — display name to render alongside.
- `payload` (object, optional) — type-specific extras (e.g. mentioned-name list, file metadata, poll question).

## Fires trigger

- [trigger/eventEmission](../../trigger/eventEmission.md)
