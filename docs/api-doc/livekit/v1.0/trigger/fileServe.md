# fileServe

Stream a previously-uploaded file off disk to the caller.

## Activated by

- input: [input/api/downloadFile](../input/api/downloadFile.md)

## What it does

1. Resolve the path: `{uploads-dir}/{meetingId}/{fileId}_*` — the trailing filename portion is matched on disk by prefix because the original sanitised name was discarded from the URL.
2. 404 if the meeting directory or the prefixed file is missing.
3. Stream the bytes with `Content-Disposition: attachment` and `Content-Type` inferred from the on-disk file.

## Produces

- (no new artifact — serves an existing [output/uploadedFile](../output/uploadedFile.md))

## Idempotency & retries

Pure read.

## Notes

permitAll route. Authorisation is implicit in the unguessable `(meetingId, fileId)` tuple — anyone with the chat message has the URL. The trade-off here is that *anyone who ever had access to the chat* keeps the URL forever, even after leaving the meeting. Acceptable for the prototype; the orchestrix-v2 port will likely add session-bound serving.
