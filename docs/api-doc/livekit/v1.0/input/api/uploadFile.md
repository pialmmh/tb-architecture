# uploadFile()

HTTP API call. `POST /api/meetings/{meetingId}/files`. Multipart upload of a file the user wants to share into the meeting chat. The response carries a `fileId` and `downloadUrl` that the client embeds into a sentinel-tagged chat message (`[meet:file]<json>`).

permitAll on the route — gating is in the controller. Hosts and `ADMIN`s may always upload; other callers may upload only if `Meeting.participantsCanShareFile = true`. Hard limits: 25 MB max file size, 30 MB max request size, blocked extensions (`exe`, `bat`, `cmd`, `com`, `scr`, `ps1`, `vbs`, `js`, `sh`, `dmg`, `app`, `msi`).

## Params

- `meetingId` (path, required)
- `file` (multipart part, required) — the file body. Filename is sanitised before storage.

## Computed (not supplied by caller)

- `fileId` — UUID prefix; final on-disk path is `{uploads-dir}/{meetingId}/{fileId}_{sanitizedName}`.
- `MENTION` / `FILE_SHARED` events are appended by the client via [emitClientEvent](emitClientEvent.md) after the chat message lands.

## Fires trigger

- [trigger/fileIngest](../../trigger/fileIngest.md)
