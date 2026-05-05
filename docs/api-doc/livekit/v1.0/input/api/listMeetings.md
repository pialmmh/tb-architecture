# listMeetings()

HTTP API call. `GET /api/meetings`. Lists meetings visible to the caller. `ADMIN` sees all rows; other roles see meetings they host or are listed on as participants.

Authenticated. Each list call also runs an auto-end pass — meetings whose `endTime` has elapsed are flipped to `ENDED` lazily here, so listings reflect current lifecycle state without a dedicated cron.

## Params

(none — caller resolved from session)

## Fires trigger

- [trigger/meetingQuery](../../trigger/meetingQuery.md)
