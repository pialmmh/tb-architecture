# updateMeeting()

HTTP API call. `PUT /api/meetings/{id}`. Mutates meeting metadata and (optionally) drives status transitions.

Authenticated. Caller must be the host or an `ADMIN`. Status transitions are validated against the lifecycle state machine — illegal transitions return 409 with body `{"error": "meeting is X — ..."}`. When status flips to `ENDED`, `actualEndedAt` is stamped and a `MEETING_ENDED` event is appended.

## Params

All fields optional; only the supplied keys are updated.

- `title`, `scheduledAt`, `endTime`
- `status` — must be a legal next state per `MeetingService.isAllowedTransition`.
- `securityMode`, `autoAdmit`, `locked`
- `maxParticipants`, `recordingEnabled`
- `muteOnEntry`, `videoOffOnEntry`
- `participantsCanShareScreen`, `participantsCanShareFile`
- `eventVisibility`

## Fires trigger

- [trigger/meetingMutation](../../trigger/meetingMutation.md)
