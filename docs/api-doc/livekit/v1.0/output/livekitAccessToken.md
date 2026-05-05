name: livekitAccessToken
type: signed JWT (HS256) — body of the `/token` response and the `livekitToken` field on admitted `pendingJoiner`
trigger: [tokenIssuance](../trigger/tokenIssuance.md), [magicLinkClaim](../trigger/magicLinkClaim.md) (direct path), [waitingRoomManagement](../trigger/waitingRoomManagement.md) (on admit)

LiveKit access token. Signed with the LK API key + secret (`livekit.api-key`, `livekit.api-secret` from `application.yml`). The browser hands it to the LK client SDK to join the room.

Notable claims:

- `room` — meeting id
- `identity` — caller's LK identity
- `name` — display name
- `metadata` — JSON: `{isHost, isAdmin, ...}` — moderation features key off this
- grants — `canPublish`, `canSubscribe`, `canPublishData`, `canPublishSources` (limits screen-share when policy says so), `recorder` for `ghost` role, `hidden` for `ghost`

TTL is short (typically 10 min) — the LK SDK refreshes via the `/token` endpoint as needed.

**Important:** the JWT is self-contained. Permissions inside it are *not* updated when meeting settings change mid-call (e.g. flipping `participantsCanShareScreen`). The UI compensates with peer-broadcast policy on a LiveKit data channel.
