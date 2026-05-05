name: recording
type: json + DB row (`recordings`)
trigger: [recordingControl](../trigger/recordingControl.md)

A single recording attempt for a meeting.

- `id` (UUID) — primary key
- `meetingId`
- `egressId` — LK Egress job id; used to issue the stop RPC
- `status` — `STARTING` | `ACTIVE` | `STOPPED` | `FAILED`
- `startedAt`, `endedAt`
- `filePath` — absolute path on disk; null until egress finalises
- `downloadUrl` — derived from `id` for the `/recordings/{id}/file` endpoint

A meeting may have multiple recordings if start/stop is exercised more than once.
