name: meetingEvent
type: in-memory ring entry (`MeetingEventService`) + SSE message
trigger: [eventEmission](../trigger/eventEmission.md), [eventReplay](../trigger/eventReplay.md) (replay only — does not produce new entries)

A single timeline entry. Same payload appears in three places: the in-memory ring buffer, the SSE stream `event: event\ndata: <json>` frames, and the JSON array returned by backfill.

- `id` (long, monotonic) — assigned at append time; the SSE / backfill cursor
- `meetingId`
- `type` — see registry below
- `actorIdentity`, `actorName` (nullable)
- `payload` (object, nullable) — type-specific extras (e.g. mention list, file metadata, deny reason)
- `ts` — server clock at append time

**Event type registry** (server-only marked `[S]`, client-emittable marked `[C]`):

```
MEETING_STARTED          [S]   PARTICIPANT_KICKED   [S]
MEETING_ENDED            [S]   PARTICIPANT_BLOCKED  [S]
PARTICIPANT_JOINED       [S]   LOCKED               [S]
PARTICIPANT_LEFT         [C]   UNLOCKED             [S]
SCREEN_SHARE_STARTED     [C]   KNOCK_DENIED         [S]
SCREEN_SHARE_STOPPED     [C]   FILE_SHARED          [C]
HAND_RAISED              [C]   MENTION              [C]
HAND_LOWERED             [C]   POLL_CREATED         [C]
RECORDING_STARTED        [S]   POLL_CLOSED          [C]
RECORDING_STOPPED        [S]
```

The whitelist of `[C]` types is `MeetingEventController.CLIENT_EMITTABLE` — anything else from a client is rejected with 400. **Storage is in-process and bounded** (`MeetingEventService.RING_CAP`); restart wipes timelines.
