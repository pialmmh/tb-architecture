# stopRecording()

HTTP API call. `POST /api/meetings/{meetingId}/recording/stop`. Ends the active recording for a meeting: stops the LK egress job, flips the `Recording` row to its terminal state, appends `RECORDING_STOPPED`. Idempotent — calling stop with no active recording returns the most recent one.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/recordingControl](../../trigger/recordingControl.md)
