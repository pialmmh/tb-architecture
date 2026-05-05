# joinMeeting()

HTTP API call. `POST /api/join/{token}`. Claims a magic link and either (a) returns a LiveKit JWT directly or (b) enrolls the joiner into the waiting room and returns a knock id to poll on.

permitAll. `@Transactional`. The branching depends on the meeting's `securityMode` and `autoAdmit` flag, plus the device fingerprint cookie set by [resolveMagicLink](resolveMagicLink.md):

- `PUBLIC` + `autoAdmit=true` → mint LK token immediately, record claim, fire `PARTICIPANT_JOINED`.
- `PUBLIC` + `autoAdmit=false`, or `PRIVATE` → enroll into waiting room, return `knockId`.
- Caller's `(meetingId, deviceFingerprint | contactChannel | userId)` matches a `BlockedUser` row → 403.

## Params

- `token` (path, required) — magic link token.
- `displayName` (string, required) — name to render for this participant.

## Computed (not supplied by caller)

- `deviceFingerprint` — read from the `meet_dev` cookie; written if absent.
- `claim` — `magic_link_claims` row binding token + device.

## Fires trigger

- [trigger/magicLinkClaim](../../trigger/magicLinkClaim.md)
