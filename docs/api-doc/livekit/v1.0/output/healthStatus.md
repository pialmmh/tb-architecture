name: healthStatus
type: json — `{ok: true}`
trigger: [healthCheck](../trigger/healthCheck.md)

The literal payload `{"ok": true}`. The shape is intentionally minimal — the response body never carries detail, even on failure (failure surfaces as a non-200 HTTP status, not as `{ok: false}`).
