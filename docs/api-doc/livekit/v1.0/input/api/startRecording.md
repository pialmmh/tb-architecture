# startRecording()

HTTP API call. `POST /api/meetings/{meetingId}/recording/start`. Begins a server-side composite recording of the LiveKit room via the LK Egress service. Inserts a `Recording` row, fires off the egress RPC, and appends `RECORDING_STARTED`.

Authenticated. Host or `ADMIN`. Returns 502 if egress refuses (e.g. LK egress service unreachable). Idempotent at the meeting scope: starting a recording while one is already active returns the active one.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/recordingControl](../../trigger/recordingControl.md)
