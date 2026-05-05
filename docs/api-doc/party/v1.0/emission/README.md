# emission/ (empty in v1.0)

Reserved for **caller-facing async streams** — websocket frames, kafka per-call topics, redis pub/sub channels keyed by a correlation id from the sync `## Response`.

Party v1.0 has none — every API returns synchronously and finishes.

When adding the first emission, follow the [apibuilder skill](../../../../ai-skills/apibuilder.md) — the source `api/<name>.md` adds an `## Async events` section listing them, and each emission file declares its delivery channel, correlation key, sequence position, and the cause-code rows it can carry.
