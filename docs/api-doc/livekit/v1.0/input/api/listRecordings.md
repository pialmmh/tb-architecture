# listRecordings()

HTTP API call. `GET /api/meetings/{meetingId}/recordings`. Returns all `Recording` rows for the meeting, including in-progress ones. Each row carries a `downloadUrl` once the file is materialised.

Authenticated.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/recordingControl](../../trigger/recordingControl.md)
