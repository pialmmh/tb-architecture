# consume/ (empty in v1.0)

Reserved for **async subscribers** — kafka topic consumers, redis pub/sub subscribers, websocket server-inbound handlers.

Party v1.0 has none — it is sync-only and does not subscribe to any external streams.

When adding the first consumer, follow the [apibuilder skill](../../../../ai-skills/apibuilder.md) §5.2 template (consume variant — declares broker, consumer-group, delivery guarantee, DLQ, replay policy).
