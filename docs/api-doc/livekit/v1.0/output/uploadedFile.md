name: uploadedFile
type: file on disk + json response
trigger: [fileIngest](../trigger/fileIngest.md)

A file shared into a meeting's chat.

**On disk**: stored at `{uploads-dir}/{meetingId}/{fileId}_{sanitizedName}`. Per-meeting subdirectory keeps file paths from colliding across meetings; the `{fileId}_` prefix ensures uniqueness within a meeting.

**Json response**: `{fileId, name, size, contentType, downloadUrl}`. The client embeds this into a `[meet:file]<json>` sentinel chat message and emits a `FILE_SHARED` event via `emitClientEvent`. The sentinel pattern lets us extend chat messages without migrating off LiveKit's built-in `useChat` hook.

There is **no garbage collection** in v1.0 — files survive meeting end. Disk planning matters for long-running deployments.
