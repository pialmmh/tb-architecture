# trigger/ (empty in v1.0)

Reserved for **internal events** that stimulate the service — timers, domain events (`SubscriptionExpired`, `TenantSuspended`), state-change notifications.

Party v1.0 has none. The directory exists so the four ingress categories (`api/`, `trigger/`, `consume/`, plus the optional `handler/`) and outgoing `emission/` are visible at a glance.

When adding the first trigger, follow the [apibuilder skill](../../../../ai-skills/apibuilder.md) §5.2 template.
