# downloadRecording()

HTTP API call. `GET /api/recordings/{id}/file`. Streams the recording file off disk with `Content-Type: video/mp4` and inline disposition. 404 if the row exists but the file is missing or the file path is null (e.g. egress still in progress).

Authenticated.

## Params

- `id` (path, required) — `Recording.id`.

## Fires trigger

- [trigger/recordingControl](../../trigger/recordingControl.md)
