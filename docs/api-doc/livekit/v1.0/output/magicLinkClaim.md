name: magicLinkClaim
type: DB row (`magic_link_claims`) + side-effect: `meet_dev` HttpOnly cookie
trigger: [magicLinkClaim](../trigger/magicLinkClaim.md)

Records that a specific device used a specific magic link.

- `(token, deviceFingerprint)` — composite primary key (one row per device per link)
- `claimedAt`, `lastSeenAt`
- `ip`, `userAgent` — captured at claim time for audit

The accompanying `meet_dev` cookie carries the device fingerprint; it's set on first `resolveMagicLink` call and read by `joinMeeting`. Charset/collation matches `magic_links` exactly so the FK is accepted (this was a launch-day bug — see V7 migration notes).
