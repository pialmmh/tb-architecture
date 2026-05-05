# setMeetingLock()

HTTP API call. `POST /api/meetings/{id}/lock`. Toggles or sets the meeting's lock flag. A locked meeting refuses new token issuances (existing participants are unaffected).

Authenticated. Host or `ADMIN`. When the lock state actually changes, a `LOCKED` or `UNLOCKED` event is appended to the timeline.

## Params

- `locked` (bool, optional) — explicit value; if omitted, toggles the current value.

## Fires trigger

- [trigger/meetingMutation](../../trigger/meetingMutation.md)
