name: pendingJoiner
type: in-memory entry (process-local map in `WaitingRoomService`)
trigger: [waitingRoomManagement](../trigger/waitingRoomManagement.md)

A waiting-room queue entry. **Not persisted** — backend restart drops every entry. Acceptable because knocks are short-lived (10 min TTL).

- `knockId` — unique id; passed back to the joiner for polling
- `meetingId`, `magicLinkToken`
- `displayName`, `deviceFingerprint`, `ip`, `userAgent`
- `state` — `PENDING` → (`ADMITTED` | `DENIED` | `TIMED_OUT`)
- `requestedAt`, `resolvedAt`
- `denyReason` (set on DENIED)
- `livekitToken` (set on ADMITTED) — minted server-side at admit time and handed to the joiner on their next status poll

Resolved entries linger ~10 min in memory so the joiner's poll loop sees the terminal state, then are evicted by the [waitingRoomSweep](../input/clock/waitingRoomSweep.md) clock.
