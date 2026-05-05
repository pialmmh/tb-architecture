# LiveKit-Meet — Versioned Input / Trigger / Output

> **Adding or changing API docs in this folder?** Use the **apibuilder** skill — it has the full convention, version rules, file templates, and worked examples.
>
> - Skill file: [`../../ai-skills/apibuilder.md`](../../ai-skills/apibuilder.md)
> - Slash invocation (where supported): `/apibuilder <version> <prompt>` — e.g. `/apibuilder v1 "add a breakout-rooms endpoint"`
>
> The summary below is enough to navigate; reach for the skill file when authoring.

---

LiveKit-Meet is the meeting-room service: it owns meeting lifecycle (scheduled → live → ended), participant rosters, magic-link invites with optional waiting room, LiveKit JWT minting, recording control, in-call moderation, file sharing, and a meeting-event timeline. The service is single-tenant by design — it is the prototype that will be folded into orchestrix-v2 as a meeting subsystem.

Each `vX.Y/` folder is a self-contained snapshot of the contract surface at that version.

```
livekit/
├── README.md                 ← you are here
└── v1.0/                     ← current
    ├── input/                what the service receives
    │   ├── api/              HTTP endpoints
    │   └── clock/            scheduled tasks
    ├── trigger/              named business actions
    └── output/               named, typed artifacts produced by triggers
```

## Current version: `v1.0`

### Inputs — HTTP API

Grouped by area; one file per endpoint.

| Area | File | Verb + Path |
|---|---|---|
| Health | [v1.0/input/api/health.md](v1.0/input/api/health.md) | GET /api/health |
| Auth | [v1.0/input/api/login.md](v1.0/input/api/login.md) | POST /api/auth/login |
| Auth | [v1.0/input/api/logout.md](v1.0/input/api/logout.md) | POST /api/auth/logout |
| Auth | [v1.0/input/api/me.md](v1.0/input/api/me.md) | GET /api/auth/me |
| Users | [v1.0/input/api/listUsers.md](v1.0/input/api/listUsers.md) | GET /api/users |
| Users | [v1.0/input/api/getUser.md](v1.0/input/api/getUser.md) | GET /api/users/{id} |
| Meetings | [v1.0/input/api/listMeetings.md](v1.0/input/api/listMeetings.md) | GET /api/meetings |
| Meetings | [v1.0/input/api/createMeeting.md](v1.0/input/api/createMeeting.md) | POST /api/meetings |
| Meetings | [v1.0/input/api/getMeeting.md](v1.0/input/api/getMeeting.md) | GET /api/meetings/{id} |
| Meetings | [v1.0/input/api/updateMeeting.md](v1.0/input/api/updateMeeting.md) | PUT /api/meetings/{id} |
| Meetings | [v1.0/input/api/setMeetingLock.md](v1.0/input/api/setMeetingLock.md) | POST /api/meetings/{id}/lock |
| Meetings | [v1.0/input/api/issueToken.md](v1.0/input/api/issueToken.md) | POST /api/meetings/{id}/token |
| Participants | [v1.0/input/api/listParticipants.md](v1.0/input/api/listParticipants.md) | GET /api/meetings/{id}/participants |
| Participants | [v1.0/input/api/addParticipant.md](v1.0/input/api/addParticipant.md) | POST /api/meetings/{id}/participants |
| Participants | [v1.0/input/api/removeParticipant.md](v1.0/input/api/removeParticipant.md) | DELETE /api/meetings/{id}/participants/{pid} |
| Invites | [v1.0/input/api/shareInvite.md](v1.0/input/api/shareInvite.md) | POST /api/meetings/{id}/invites/share |
| Invites | [v1.0/input/api/listInvites.md](v1.0/input/api/listInvites.md) | GET /api/meetings/{id}/invites |
| Invites | [v1.0/input/api/createInvite.md](v1.0/input/api/createInvite.md) | POST /api/meetings/{id}/invites |
| Invites | [v1.0/input/api/revokeInvite.md](v1.0/input/api/revokeInvite.md) | DELETE /api/meetings/{id}/invites/{token} |
| Join | [v1.0/input/api/resolveMagicLink.md](v1.0/input/api/resolveMagicLink.md) | GET /api/magic/{token} |
| Join | [v1.0/input/api/joinMeeting.md](v1.0/input/api/joinMeeting.md) | POST /api/join/{token} |
| Join | [v1.0/input/api/knockStatus.md](v1.0/input/api/knockStatus.md) | GET /api/join/{token}/knock/{knockId} |
| Waiting Room | [v1.0/input/api/listPending.md](v1.0/input/api/listPending.md) | GET /api/meetings/{id}/pending |
| Waiting Room | [v1.0/input/api/admitKnock.md](v1.0/input/api/admitKnock.md) | POST /api/meetings/{id}/pending/{knockId}/admit |
| Waiting Room | [v1.0/input/api/denyKnock.md](v1.0/input/api/denyKnock.md) | POST /api/meetings/{id}/pending/{knockId}/deny |
| Recording | [v1.0/input/api/startRecording.md](v1.0/input/api/startRecording.md) | POST /api/meetings/{id}/recording/start |
| Recording | [v1.0/input/api/stopRecording.md](v1.0/input/api/stopRecording.md) | POST /api/meetings/{id}/recording/stop |
| Recording | [v1.0/input/api/listRecordings.md](v1.0/input/api/listRecordings.md) | GET /api/meetings/{id}/recordings |
| Recording | [v1.0/input/api/downloadRecording.md](v1.0/input/api/downloadRecording.md) | GET /api/recordings/{id}/file |
| Moderation | [v1.0/input/api/muteParticipant.md](v1.0/input/api/muteParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/mute |
| Moderation | [v1.0/input/api/stopVideoParticipant.md](v1.0/input/api/stopVideoParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/stop-video |
| Moderation | [v1.0/input/api/kickParticipant.md](v1.0/input/api/kickParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/kick |
| Moderation | [v1.0/input/api/blockParticipant.md](v1.0/input/api/blockParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/block |
| Moderation | [v1.0/input/api/promoteParticipant.md](v1.0/input/api/promoteParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/promote |
| Moderation | [v1.0/input/api/demoteParticipant.md](v1.0/input/api/demoteParticipant.md) | POST /api/meetings/{id}/moderate/{identity}/demote |
| Moderation | [v1.0/input/api/muteAll.md](v1.0/input/api/muteAll.md) | POST /api/meetings/{id}/moderate/mute-all |
| Moderation | [v1.0/input/api/setScreenSharePolicy.md](v1.0/input/api/setScreenSharePolicy.md) | POST /api/meetings/{id}/moderate/screen-share |
| Moderation | [v1.0/input/api/endMeetingForAll.md](v1.0/input/api/endMeetingForAll.md) | POST /api/meetings/{id}/moderate/end |
| Files | [v1.0/input/api/uploadFile.md](v1.0/input/api/uploadFile.md) | POST /api/meetings/{id}/files |
| Files | [v1.0/input/api/downloadFile.md](v1.0/input/api/downloadFile.md) | GET /api/files/{meetingId}/{fileId} |
| Events | [v1.0/input/api/backfillEvents.md](v1.0/input/api/backfillEvents.md) | GET /api/meetings/{id}/events |
| Events | [v1.0/input/api/streamEvents.md](v1.0/input/api/streamEvents.md) | GET /api/meetings/{id}/events/stream (SSE) |
| Events | [v1.0/input/api/emitClientEvent.md](v1.0/input/api/emitClientEvent.md) | POST /api/meetings/{id}/events/client |

### Inputs — Clock

| File | Cadence |
|---|---|
| [v1.0/input/clock/waitingRoomSweep.md](v1.0/input/clock/waitingRoomSweep.md) | every 60s — expire stale knocks, evict resolved entries |

### Triggers

| File | Activated by |
|---|---|
| [v1.0/trigger/healthCheck.md](v1.0/trigger/healthCheck.md) | health |
| [v1.0/trigger/session.md](v1.0/trigger/session.md) | login, logout, me |
| [v1.0/trigger/userDirectory.md](v1.0/trigger/userDirectory.md) | listUsers, getUser |
| [v1.0/trigger/meetingProvisioning.md](v1.0/trigger/meetingProvisioning.md) | createMeeting |
| [v1.0/trigger/meetingMutation.md](v1.0/trigger/meetingMutation.md) | updateMeeting, setMeetingLock |
| [v1.0/trigger/meetingQuery.md](v1.0/trigger/meetingQuery.md) | listMeetings, getMeeting |
| [v1.0/trigger/tokenIssuance.md](v1.0/trigger/tokenIssuance.md) | issueToken |
| [v1.0/trigger/participantRosterManagement.md](v1.0/trigger/participantRosterManagement.md) | listParticipants, addParticipant, removeParticipant |
| [v1.0/trigger/magicLinkLifecycle.md](v1.0/trigger/magicLinkLifecycle.md) | shareInvite, listInvites, createInvite, revokeInvite |
| [v1.0/trigger/magicLinkClaim.md](v1.0/trigger/magicLinkClaim.md) | resolveMagicLink, joinMeeting, knockStatus |
| [v1.0/trigger/waitingRoomManagement.md](v1.0/trigger/waitingRoomManagement.md) | listPending, admitKnock, denyKnock, waitingRoomSweep |
| [v1.0/trigger/recordingControl.md](v1.0/trigger/recordingControl.md) | startRecording, stopRecording, listRecordings, downloadRecording |
| [v1.0/trigger/moderationAction.md](v1.0/trigger/moderationAction.md) | mute, stop-video, kick, block, promote, demote, mute-all, screen-share, end |
| [v1.0/trigger/fileIngest.md](v1.0/trigger/fileIngest.md) | uploadFile |
| [v1.0/trigger/fileServe.md](v1.0/trigger/fileServe.md) | downloadFile |
| [v1.0/trigger/eventEmission.md](v1.0/trigger/eventEmission.md) | emitClientEvent + chained from many server triggers |
| [v1.0/trigger/eventReplay.md](v1.0/trigger/eventReplay.md) | backfillEvents, streamEvents |

### Outputs

| File | Type |
|---|---|
| [v1.0/output/healthStatus.md](v1.0/output/healthStatus.md) | json |
| [v1.0/output/sessionCookie.md](v1.0/output/sessionCookie.md) | http cookie + json |
| [v1.0/output/userView.md](v1.0/output/userView.md) | json |
| [v1.0/output/meeting.md](v1.0/output/meeting.md) | json + DB row (`meetings`) |
| [v1.0/output/meetingParticipant.md](v1.0/output/meetingParticipant.md) | json + DB row (`meeting_participants`) |
| [v1.0/output/magicLink.md](v1.0/output/magicLink.md) | json + DB row (`magic_links`) |
| [v1.0/output/magicLinkClaim.md](v1.0/output/magicLinkClaim.md) | DB row (`magic_link_claims`) + http cookie |
| [v1.0/output/pendingJoiner.md](v1.0/output/pendingJoiner.md) | in-memory entry |
| [v1.0/output/livekitAccessToken.md](v1.0/output/livekitAccessToken.md) | signed JWT |
| [v1.0/output/recording.md](v1.0/output/recording.md) | json + DB row (`recordings`) |
| [v1.0/output/recordingFile.md](v1.0/output/recordingFile.md) | mp4 file on disk |
| [v1.0/output/uploadedFile.md](v1.0/output/uploadedFile.md) | file on disk + json |
| [v1.0/output/meetingEvent.md](v1.0/output/meetingEvent.md) | in-memory ring entry + SSE message |
| [v1.0/output/blockedUser.md](v1.0/output/blockedUser.md) | DB row (`blocked_users`) |

## How the three relate

```
input  ─fires→  trigger  ─produces→  output
                   ↑                    ↑
              (also fired by         (output may have
               clock or another       multiple triggers)
               trigger)
```

- An **input** alone never produces an output directly — it fires a trigger, the trigger does the work.
- A **trigger** is the unit of business behaviour. It has activation sources (one or more) and produces artifacts (one or more).
- An **output** records its trigger(s) — the audit answer to "what caused this artifact to exist?".

Cross-cutting note: `meetingEvent` is produced by **most** triggers in this service, not just the obvious "emit event" path. Lifecycle transitions, joins, kicks, blocks, recording start/stop, knock denies — all chain into the eventEmission trigger as a side-effect. The output file lists the full producer set.

## Per-file conventions (summary)

- **input/** files: what the call looks like + which trigger(s) it fires.
- **trigger/** files: activation sources, what it does, what it produces, idempotency + retries.
- **output/** files: `name`, `type`, `trigger(s)`, brief description.
- camelCase action names; long + self-descriptive over short + cryptic.
- **Cross-link within the same version only.** Don't link `v1.1/` → `v1.0/`; copy the file if it's reused.

Full templates with section-by-section structure live in the skill file.

## Versioning rules (summary)

| Change | Bump |
|---|---|
| New input / trigger / output | MINOR |
| New optional field on an existing one | MINOR |
| Removed input / renamed trigger / changed output shape | MAJOR |
| Added required field to an existing input | MAJOR |
| Typo, prose clarification | no bump |

- Bumping = **copy the latest version dir to a new `vX.Y/` and edit there**. Don't delta-edit; each version must be self-contained.
- A version dir **freezes once a release using it ships**. Edits after that are limited to clarifications that don't change the wire contract.
- The "current" version is whichever directory name is highest. No symlinks.

## Known gaps in v1.0

These are documented honestly in the relevant trigger files so future readers know what isn't there:

- **Token-mint participant gate** is half-implemented — `issueToken` doesn't yet require the caller to be in the meeting roster. Tracked in the source repo's security backlog.
- **OTP delivery for PRIVATE magic links** (email + SMS) is referenced in the migration history but not built. The endpoints `requestOtp` / `verifyOtp` are absent from v1.0; expect them in v1.1.
- **Vote integrity for in-call polls** — polls are peer-distributed via LiveKit data channels, not stored in the backend, so the server has no view of them and ballot-stuffing is possible in PUBLIC meetings.

## Out of scope for this folder

- **Setup / install runbook** (LiveKit server install, MariaDB Flyway bootstrap, Vite dev, network ports) — that lives in the livekit-meet source repo, not here. This folder is the outside-in contract view.
- **Peer-to-peer LiveKit data-channel topics** (raise-hand, reactions, polls, pins, meeting-policy) — these are room-level peer state and never touch the backend, so they don't appear in the input/trigger/output graph.
- **UI component internals** — React hooks, MUI overrides, drawer layout. Source-repo concerns.
