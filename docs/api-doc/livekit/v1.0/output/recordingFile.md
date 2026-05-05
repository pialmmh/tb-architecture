name: recordingFile
type: mp4 file on disk
trigger: [recordingControl](../trigger/recordingControl.md)

The composite room recording produced by LK Egress. Created **asynchronously** — the `Recording` row exists from `start`, but `filePath` and the file itself only appear when egress finalises (some seconds to minutes after `stop`, depending on duration and load).

Storage location is configured in LK Egress's own config, not in livekit-meet's `application.yml`. The path is reported back to livekit-meet by Egress and persisted on the `Recording` row.

Served by [downloadRecording](../input/api/downloadRecording.md). 404 until the path is populated.
