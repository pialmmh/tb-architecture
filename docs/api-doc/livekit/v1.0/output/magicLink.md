name: magicLink
type: json + DB row (`magic_links`)
trigger: [magicLinkLifecycle](../trigger/magicLinkLifecycle.md)

A token-bearing invitation to join a meeting. Two variants discriminated by `linkType`:

- `PUBLIC` — one per meeting; `contactChannel` and `participantId` null. Default 30-day TTL.
- `PRIVATE` — per-recipient; `contactChannel` is the email it was sent to. Default 7-day TTL.

Other notable fields:

- `token` (varchar(64), primary key) — the random string in the URL.
- `meetingId`
- `createdAt`, `expiresAt`
- claims (joined via `magic_link_claims`) — which device fingerprints have already used this link.

Schema is current as of migration V7 (link redesign).
