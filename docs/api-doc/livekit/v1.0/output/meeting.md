name: meeting
type: json + DB row (`meetings`)
trigger: [meetingProvisioning](../trigger/meetingProvisioning.md), [meetingMutation](../trigger/meetingMutation.md), [meetingQuery](../trigger/meetingQuery.md), [moderationAction](../trigger/moderationAction.md) (setScreenSharePolicy + endMeetingForAll)

The meeting record. Same value shape returned in JSON responses and persisted to MariaDB. Key fields:

- `id` (UUID, primary key, also LK `roomName`)
- `title`, `hostUserId`
- `scheduledAt`, `startedAt`, `endTime`, `actualEndedAt`
- `status` — state machine: `SCHEDULED → LIVE → ENDED` (or `→ CANCELLED`). Terminal states reject lifecycle mutations with 409.
- `securityMode` — `PUBLIC` | `PRIVATE`. Selects the magic-link kind.
- `autoAdmit` — PUBLIC-only knob; gates the waiting room.
- `locked` — when true, blocks new token issuance for non-host callers.
- `maxParticipants` (default 50)
- `recordingEnabled`, `muteOnEntry`, `videoOffOnEntry`, `participantsCanShareScreen`, `participantsCanShareFile`
- `eventVisibility` — `PUBLIC` | `HOST_ONLY`; gates the event timeline endpoints.

Schema is current as of migration V12 (`participantsCanShareFile`).
