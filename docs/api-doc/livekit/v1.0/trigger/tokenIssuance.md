# tokenIssuance

Mint a LiveKit JWT for a joiner. Also where meeting lifecycle is "armed" — the first non-ghost join is what flips a `SCHEDULED` meeting into `LIVE`.

## Activated by

- input: [input/api/issueToken](../input/api/issueToken.md)
- chained from: [trigger/magicLinkClaim](./magicLinkClaim.md) — guests joining via magic link don't hit `/token` directly; the claim flow mints for them.
- chained from: [trigger/waitingRoomManagement](./waitingRoomManagement.md) — admit-from-waiting-room mints token server-side and stores it on the pending entry.

## What it does

1. Load meeting; 404 if missing; 409 if status is terminal.
2. Reject if `Meeting.locked` is true and the caller isn't host / admin.
3. Reject if `Meeting.maxParticipants` would be exceeded by this join.
4. If the meeting is `SCHEDULED` and the requested role isn't `ghost`, mark it `LIVE` and stamp `startedAt`.
5. Build the LK access token: identity, name, room = `meeting.id`, grants based on `role` + meeting settings (publish, subscribe, screen-share, recording, data, hidden for `ghost`).
6. Append `PARTICIPANT_JOINED` to the timeline.

## Produces

- [output/livekitAccessToken](../output/livekitAccessToken.md)
- chained: [trigger/eventEmission](./eventEmission.md) — `PARTICIPANT_JOINED`, and `MEETING_STARTED` if we just transitioned to `LIVE`.

## Idempotency & retries

Each call mints a fresh JWT — the LiveKit side doesn't care about duplicates, but the timeline picks up duplicate `PARTICIPANT_JOINED` events on retries. The UI dedupes by identity client-side.

## Notes

**Known authz gap (S1):** the trigger does not yet require the caller to be in `meeting_participants` for the meeting. Currently any authenticated user who knows the meeting id can mint a token. Tightening this is on the security backlog. The hole is documented in `MeetingController.issueToken` near the `// TODO(security)` marker.

Permissions are baked into the JWT at mint time; mid-call setting changes (e.g. flipping `participantsCanShareScreen`) do **not** retroactively rewrite tokens. The UI compensates by listening on a peer-broadcast `meeting-policy` topic.
