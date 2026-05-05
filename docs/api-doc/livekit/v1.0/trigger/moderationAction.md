# moderationAction

Host-side controls that act on the live LK room: mute, kick, block, promote / demote, mute-all, screen-share policy flip, and meeting end-for-all. All require host or `ADMIN`.

## Activated by

- input: [input/api/muteParticipant](../input/api/muteParticipant.md)
- input: [input/api/stopVideoParticipant](../input/api/stopVideoParticipant.md)
- input: [input/api/kickParticipant](../input/api/kickParticipant.md)
- input: [input/api/blockParticipant](../input/api/blockParticipant.md)
- input: [input/api/promoteParticipant](../input/api/promoteParticipant.md)
- input: [input/api/demoteParticipant](../input/api/demoteParticipant.md)
- input: [input/api/muteAll](../input/api/muteAll.md)
- input: [input/api/setScreenSharePolicy](../input/api/setScreenSharePolicy.md)
- input: [input/api/endMeetingForAll](../input/api/endMeetingForAll.md)

## What it does

Each action is a thin wrapper over a LiveKit server-side API call (mute by track SID or by kind, remove participant, update permissions / metadata) plus optional DB side-effects:

- **mute / stop-video**: scan or single-track mute via `LiveKitRoomService.mutePublishedTrack` / `muteByKind`.
- **kick**: `LiveKitRoomService.removeParticipant`. Append `PARTICIPANT_KICKED`.
- **block**: insert `BlockedUser` row keyed on whatever identifier is supplied (`userId | contactChannel | deviceFingerprint`); then kick. Append `PARTICIPANT_BLOCKED`.
- **promote / demote**: re-fetch the participant from LK, flip `isHost` in metadata, push updated permissions.
- **mute-all**: iterate all participants; skip those carrying `isHost` or `isAdmin` in metadata; mute audio tracks. Best-effort — per-participant failures are swallowed.
- **setScreenSharePolicy**: persist the new flag on `Meeting`. Does *not* touch already-issued LK tokens (JWT-baked).
- **endMeetingForAll**: kick everybody, flip meeting `status=ENDED`, stamp `actualEndedAt`, append `MEETING_ENDED`, clear the meeting's event ring buffer.

## Produces

- [output/blockedUser](../output/blockedUser.md) (block)
- [output/meeting](../output/meeting.md) (setScreenSharePolicy, endMeetingForAll)
- chained: [trigger/eventEmission](./eventEmission.md) — `PARTICIPANT_KICKED`, `PARTICIPANT_BLOCKED`, `MEETING_ENDED`

## Idempotency & retries

- mute / stop-video: re-running on an already-muted track is a no-op.
- kick: removing an absent participant is a no-op.
- block: `BlockedUser` insert may collide on retry; treat as success.
- promote / demote: idempotent — pushing the same metadata + perms twice is harmless.
- mute-all: idempotent in aggregate, even though individual mute calls may have already been applied.
- end-for-all: idempotent — repeated calls converge on `ENDED`.

## Notes

Promote / demote act only on the live LK session. The roster (`meeting_participants`) is **not** mutated — the participant's role on subsequent joins still derives from their original token request. This is deliberate: it keeps "I made you a co-host for this call" from leaking into the next call.
