# magicLinkClaim

The guest join flow. Resolves a magic link, optionally routes through the waiting room, and ultimately mints a LiveKit token bound to a device fingerprint.

## Activated by

- input: [input/api/resolveMagicLink](../input/api/resolveMagicLink.md) — read path; sets the device cookie
- input: [input/api/joinMeeting](../input/api/joinMeeting.md) — claim + branch
- input: [input/api/knockStatus](../input/api/knockStatus.md) — poll loop while in the waiting room

## What it does

1. **resolve**: load `magic_links` row by token; 404 if missing or expired. Set `meet_dev` HttpOnly cookie if not already present. Return meeting + invite metadata.
2. **join** (`@Transactional`): re-validate the link. Read device fingerprint from cookie. Check `BlockedUser` against `(meetingId, deviceFingerprint | contactChannel | userId)`; reject with 403 on match. Branch on `Meeting.securityMode` + `autoAdmit`:
   - **direct path** (PUBLIC + autoAdmit) — chain into [tokenIssuance](./tokenIssuance.md), record `magic_link_claims` row, append `PARTICIPANT_JOINED`, return JWT.
   - **knock path** (PUBLIC + !autoAdmit, or PRIVATE) — chain into [waitingRoomManagement](./waitingRoomManagement.md), return `knockId` for the joiner to poll.
3. **knockStatus**: look up the in-memory `PendingJoiner` by id; return its current state. On `ADMITTED`, attach the LK token that was minted at admit-time and write the `magic_link_claims` row binding token + device.

## Produces

- [output/livekitAccessToken](../output/livekitAccessToken.md) (direct path + on admission)
- [output/magicLinkClaim](../output/magicLinkClaim.md) (binds device to link)
- [output/pendingJoiner](../output/pendingJoiner.md) (knock path)
- chained: [trigger/eventEmission](./eventEmission.md) — `PARTICIPANT_JOINED` (direct + on admission)

## Idempotency & retries

`resolve` is a pure read. `join` is **not** idempotent on direct path (each call mints a fresh JWT and may double-emit `PARTICIPANT_JOINED` — the UI dedupes by identity). `join` is idempotent on knock path against an existing PENDING entry (returns the same `knockId`). `knockStatus` is a pure read.

## Notes

The device-fingerprint cookie is the single moving part that makes the device-level block stick. If a guest clears cookies they get a fresh fingerprint and the device-level block is bypassed (the contact-channel and user-id components of `BlockedUser` are still in force). This is acknowledged.

`PRIVATE` magic links are intended to require an OTP round-trip before reaching the join path; that's the **S4** work and isn't yet wired in v1.0.
