# userDirectory

Read-only user search and fetch. Backs the participant-picker on the meeting create / edit form and the host-side roster UI.

## Activated by

- input: [input/api/listUsers](../input/api/listUsers.md)
- input: [input/api/getUser](../input/api/getUser.md)

## What it does

- **listUsers**: substring-matches `q` against name / username / email. Optional `role` narrows the result. Cap 50 rows. Returns the projected `UserView` shape, never raw `User` (no password hash, no internal fields).
- **getUser**: single-row lookup by id. Same projection.

## Produces

- [output/userView](../output/userView.md)

## Idempotency & retries

Pure reads.

## Notes

The 50-row cap is arbitrary but enforced — the picker is type-ahead, not a paginated list. `CLIENT`-role callers are rejected with 403 because the directory is an internal-staff surface, not a customer-facing API.
