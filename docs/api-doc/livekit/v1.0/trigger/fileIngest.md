# fileIngest

Accept a file upload, store it on disk under the meeting's directory, return a download URL.

## Activated by

- input: [input/api/uploadFile](../input/api/uploadFile.md)

## What it does

1. Authorise: host / `ADMIN` always; other callers only if `Meeting.participantsCanShareFile = true`. Else 403.
2. Validate: ≤ 25 MB, extension not in the blocked list (`exe`, `bat`, `cmd`, `com`, `scr`, `ps1`, `vbs`, `js`, `sh`, `dmg`, `app`, `msi`).
3. Sanitise the filename (drop path separators, control chars).
4. Generate a UUID `fileId`. Write the bytes to `{uploads-dir}/{meetingId}/{fileId}_{sanitizedName}`. Create the per-meeting directory if missing.
5. Return `{fileId, name, size, contentType, downloadUrl}`. The client embeds this into a `[meet:file]<json>` chat message and emits a `FILE_SHARED` event via [emitClientEvent](../input/api/emitClientEvent.md).

## Produces

- [output/uploadedFile](../output/uploadedFile.md)

## Idempotency & retries

Not idempotent — every upload writes a new file with a fresh `fileId`. A retry after a network drop may leave an orphan on disk; there is no garbage collection in v1.0.

## Notes

`uploads-dir` is configured in `application.yml` (`app.uploads-dir`) — defaults to `/var/lib/livekit-meet/uploads`. The directory must be writable by the JVM process. Upload throughput is gated by Spring's multipart settings (`spring.servlet.multipart.max-file-size` = 25 MB, `max-request-size` = 30 MB).
