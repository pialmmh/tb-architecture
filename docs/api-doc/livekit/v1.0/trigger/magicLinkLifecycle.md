# magicLinkLifecycle

CRUD for magic links — the `MagicLink` rows that joiners present via the `/m/{token}` URL. Two kinds: `PUBLIC` (one shared link per meeting) and `PRIVATE` (per-recipient, bound to a contact channel).

## Activated by

- input: [input/api/shareInvite](../input/api/shareInvite.md) — get-or-create the PUBLIC link
- input: [input/api/listInvites](../input/api/listInvites.md)
- input: [input/api/createInvite](../input/api/createInvite.md) — create one PRIVATE link
- input: [input/api/revokeInvite](../input/api/revokeInvite.md)

## What it does

- **share**: only legal for `PUBLIC` meetings. Returns the existing PUBLIC link if one exists and isn't expired; otherwise creates one. Default TTL 30 days.
- **list**: returns every `MagicLink` row for the meeting, newest first, with their claim history.
- **create**: only legal for `PRIVATE` meetings. Inserts a new PRIVATE link bound to `invitedEmail`. Default TTL 7 days.
- **revoke**: deletes the link row. Already-issued LK tokens (which are JWT-baked and self-contained) are unaffected.

All four guard against terminal-state meetings with 409.

## Produces

- [output/magicLink](../output/magicLink.md)

## Idempotency & retries

`share` is idempotent (returns the same row each call until expiry). `create` is not (every call inserts a new PRIVATE link). `revoke` is idempotent.

## Notes

There is no email/SMS *delivery* of PRIVATE links yet — the contract returns the token; sending it to the recipient is the caller's job. OTP verification at claim time (S4) is pending.
