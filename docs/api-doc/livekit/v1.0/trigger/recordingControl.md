# recordingControl

Composite room recording via the LiveKit Egress service. Owns the `Recording` row lifecycle and the file-on-disk artifact.

## Activated by

- input: [input/api/startRecording](../input/api/startRecording.md)
- input: [input/api/stopRecording](../input/api/stopRecording.md)
- input: [input/api/listRecordings](../input/api/listRecordings.md)
- input: [input/api/downloadRecording](../input/api/downloadRecording.md)

## What it does

- **start**: insert a `Recording` row in `STARTING` state. RPC to LK Egress to begin a room composite egress targeting an on-disk path. On RPC failure, mark the row `FAILED` and return 502. Append `RECORDING_STARTED`. Idempotent at the meeting scope — starting while one is active returns the active row.
- **stop**: locate the active recording. RPC to LK Egress to stop. Flip the row to its terminal state. Append `RECORDING_STOPPED`. Idempotent — calling stop with no active recording returns the most recent row.
- **list**: read all recordings for the meeting.
- **download**: stream the file off disk. 404 if the row's file path is null (egress still finalising) or the file is missing.

## Produces

- [output/recording](../output/recording.md)
- [output/recordingFile](../output/recordingFile.md) (asynchronously, when egress finalises)
- chained: [trigger/eventEmission](./eventEmission.md) — `RECORDING_STARTED` and `RECORDING_STOPPED`

## Idempotency & retries

start and stop are idempotent at meeting scope (described above). The DB write happens before the egress RPC, so a network failure after the RPC may leave the row in a transient state until the next stop / status reconciliation.

## Notes

LiveKit Egress is a separate process — failures there surface as 502 to the API client. There is no automatic retry of the egress RPC; the user must explicitly start again.
