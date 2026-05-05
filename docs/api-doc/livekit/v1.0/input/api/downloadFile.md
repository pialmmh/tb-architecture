# downloadFile()

HTTP API call. `GET /api/files/{meetingId}/{fileId}`. Streams a file off disk with attachment disposition. Used by every chat client to pull files referenced in `[meet:file]` sentinel messages.

permitAll. Authorisation is implicit in the unguessable `(meetingId, fileId)` tuple — anyone with the chat message has the URL. 404 if the file or its parent meeting directory is missing.

## Params

- `meetingId` (path, required)
- `fileId` (path, required)

## Fires trigger

- [trigger/fileServe](../../trigger/fileServe.md)
