---
name: apibuilder
description: Create or extend versioned API-contract documentation in the flat-sibling api/trigger/consume/handler/output/emission model used across telcobright services.
invocation: /apibuilder <version> <prompt>
example: /apibuilder v1 "add an access control list for keycloak client"
---

# apibuilder — versioned API-contract authoring skill

Use this skill whenever the task is to **add, change, or version-bump documentation** for a service's API contract under `architecture/docs/api-doc/<service>/`. The skill is also valid as a standalone reading reference — point another agent at this file and they will know the conventions without any extra context.

> **Canonical live example:** `/home/mustafa/telcobright-projects/architecture/docs/api-doc/party/` — read it before authoring; copy its shape, don't invent a new one.

---

## 0. Hard rule — every API returns a value

There is no fire-and-forget API. Every endpoint under `api/` — REST, gRPC, or internal Java — returns the same envelope:

```
{
  success:       bool,                 // hard pass/fail flag
  causeCode:     int | string,         // wire value (HTTP status, custom code, or pass-through)
  causeProtocol: string,               // "HTTP" | "<SERVICE>" | "SIP" | "SS7" | "GRPC" | ...
  description:   string,               // override_description if registered, else description
  payload:       <api-specific> | null // present on success; may be null on failure
}
```

Per-protocol expectations:

- **REST** uses **standard HTTP status codes** (200, 201, 400, 401, 403, 404, 409, 422, 500, 503, …). The body still carries the envelope above; `causeCode` mirrors the HTTP status, `causeProtocol = "HTTP"`. Description is overridable via the service's cause-code registry.
- **gRPC / Java** uses the service's **custom cause-code registry** (`causeProtocol = "<SERVICE>"` by convention). Description is overridable.
- **Pass-through.** When an upstream returns a standard protocol code (SIP `486`, SS7 cause `34`, etc.), value and `causeProtocol` propagate verbatim. We attach an override description only if one is registered.
- **Single registry per service.** Every cause code referenced in a `## Response` table must exist as a row in that version's `cause-codes.md` registry. See §5.4.

---

## 1. Invocation

```
/apibuilder <version> <prompt>
```

- `<version>` — the target version directory the work goes into. Forms accepted:
  - `v1` (resolves to the highest `v1.X/` present, e.g. `v1.0`)
  - `v1.0`, `v1.1`, `v2.0`, … (exact)
  - `latest` (resolves to the highest version present)
  - `next` (start a new version dir; see §3 for when to do this)
- `<prompt>` — natural-language description of what to add/change. The skill maps the prompt onto the correct ingress/output/emission node(s).

**Examples**
- `/apibuilder v1 "add an access control list for keycloak client"`
- `/apibuilder v1.0 "expose a partner-search input that returns a partner list"`
- `/apibuilder next "breaking: createTenant now requires regionCode"`

If the user just wants the conventions and is not invoking the skill, treat this file as **read-only reference docs** — explain, don't generate.

---

## 2. The model — flat-sibling categories at the version-dir root

Each version dir uses **flat sibling categories** — no `input/` wrapper. Three ingress kinds, two artifact kinds, one optional handler layer, one registry:

```
<service>/
└── vMAJOR.MINOR/
    ├── cause-codes.md   per-version cause-code registry (§5.4)
    ├── api/             sync caller (REST/gRPC/Java); has `## Response` (+ optional `## Async events`)
    ├── trigger/         internal event — timer, domain event, state change (e.g. SubscriptionExpired)
    ├── consume/         async subscriber — kafka topic, redis pubsub, ws server inbound
    ├── handler/         OPTIONAL — named business action shared by 2+ ingresses (§2.2)
    ├── emission/        caller-facing async stream — ws / kafka / redis, correlated to a sync `## Response`
    └── output/          internal artifact — db row, internal kafka msg, file, log
```

Categories that have no entries for a version still get an empty directory with a one-line `README.md` so the convention is visible at a glance.

The default flow is two-layer:

```
ingress  ─produces→  output                          (artifacts: db row, kafka msg, file)
(api / trigger / consume)
   │
   └ (api only) ─returns→  ## Response section       (sync envelope to caller)
   └ (api only) ─emits──→  emission                  (caller-facing async stream)
```

When two or more ingresses share the same multi-step business logic, a `handler/` file is introduced to hold the shared steps:

```
ingress ─fires→ handler ─produces→ output            (3-layer; handler centralises shared logic)
```

### 2.1 The three ingress kinds

| Kind        | Examples                                                | Has `## Response`? | Has `## Async events`? |
|-------------|---------------------------------------------------------|--------------------|------------------------|
| **api**     | REST endpoint, gRPC method, internal Java call          | Yes                | Optionally             |
| **trigger** | timer / cron, `SubscriptionExpired`, `WeatherChanged`, `BalanceLow`, any internal/domain event | No                 | No                     |
| **consume** | kafka subscribe, redis subscribe, ws server inbound     | No                 | No                     |

Every ingress file declares its `Payload` (or `Params`), its `Steps` (or, when a handler exists, `Fires handler`), `Produces`, and `Idempotency`. The `## Response` section is reserved for `api/*` because only sync callers receive a return value.

### 2.2 When to use the optional `handler/` layer

Skip the layer (default — inline `## Steps`/`## Produces`/`## Idempotency` into the ingress file) when:
- Each ingress has self-contained logic.
- No reuse exists across ingresses.
- The doc stays small (the canonical example: party).

Add a `handler/<name>.md` when **at least one** of these is true:
- A multi-step process is activated by **more than one ingress** (e.g. `tenantReprovision()` invoked from both an api call and a periodic timer — the steps belong in `handler/` so they aren't duplicated).
- An async stimulus (`trigger/*` or `consume/*`) needs to run the same business flow as a sync `api/*`.
- The steps grow past ~10 items, or a sub-flow is reusable on its own.

If the layer is added, each ingress's `## Steps`/`## Produces`/`## Idempotency` collapses into a single `## Fires handler` line linking to `handler/<name>.md`. The handler then carries those sections.

Per-file rules:
- **api/** — describes the call shape, includes `## Steps` (or `## Fires handler`), `## Produces`, `## Idempotency`, **and** a `## Response` section with success + failure rows (one file per API; an API does one task, the doc stays small). Optionally `## Async events` listing emissions.
- **trigger/, consume/** — same shape minus `## Response` and `## Async events`: these inputs have no synchronous caller. Trigger declares its `Source`; consume declares broker, consumer-group, delivery guarantee, DLQ, replay policy.
- **handler/** (optional) — `## Activated by` (lists ingresses across all three kinds, plus chained handlers), `## Steps`, `## Produces`, `## Idempotency`, `## Notes`.
- **output/** — `name`, `type`, `produced by:` (links to the ingress or handler that produces it), brief description.
- **emission/** — `name`, `type`, `produced by:`, `delivery:` (channel list), `correlation:` (the key tying it to a sync response), `sequence-position:`, plus a cause-codes-carried table.
- **`cause-codes.md`** — single per-version registry of all cause codes the service emits. See §5.4.

### 2.3 Response vs output vs emission

Three distinct concepts that look superficially similar:

| Concept          | Direction      | Timing                                     | Where it lives                      |
|------------------|----------------|--------------------------------------------|-------------------------------------|
| `## Response`    | api → caller   | synchronous, exactly one per call          | inside the api file                 |
| `output/<x>.md`  | ingress → side | asynchronous artifact (db / internal feed) | `output/` dir                       |
| `emission/<x>.md`| api → caller   | async stream after the sync response       | `emission/` dir, listed in api's `## Async events` |

### 2.4 Response vs output (table)

A `## Response` section on an input doc and an `output/` file are different things:

| | output/<name>.md                                  | `## Response` on api/<name>.md        |
|---|---|---|
| Produced by         | the ingress (or its handler, when one exists)      | the API call itself                   |
| Timing              | asynchronous; a single call can emit many          | synchronous; exactly one per call     |
| Examples            | DB row, Kafka msg, file, log line                  | success/failure envelope + cause code |
| Cause-code linkage  | n/a (artifact, not a return value)                 | every row references `cause-codes.md` |

Outputs that double as the response payload are linked from the response's "payload" column (e.g., `payload: [output/tenant](../output/tenant.md)`).

---

## 3. Versioning

`vMAJOR.MINOR/` directories. Each is a self-contained snapshot.

| Change | Bump |
|---|---|
| New api / trigger / consume / handler / output / emission | MINOR |
| New optional field on existing ingress | MINOR |
| New cause-code row in `cause-codes.md` | MINOR |
| New cause-code row added to an api's `## Response` table | MINOR |
| Promoting inlined steps to a `handler/` file (no wire change) | no bump |
| Demoting a single-ingress handler back into its ingress (no wire change) | no bump |
| Removed ingress / renamed handler / changed output shape | MAJOR |
| Added required field to existing ingress | MAJOR |
| Removed cause-code row, or changed its `value`/`protocol` | MAJOR |
| Removed a cause-code row from an api's `## Response` (caller-visible) | MAJOR |
| Edited `description` / `override_description` text | no bump |
| Edited `## Steps` text without changing the wire contract | no bump |
| Typo, prose clarification | no bump |

Rules:
- **A version dir freezes once a release using it ships.** Edits after that are limited to clarifications that don't change the wire contract.
- To start a new version, **copy the latest version dir wholesale** to the new name and edit there. Don't delta-edit; the new version must be self-contained.
- Cross-links **stay within the same version**. If `v1.1` reuses a `v1.0` contract verbatim, copy the file across. Never link `v1.1/...` → `v1.0/...`.
- The "current" version is whichever folder name is highest. No symlinks.
- `next` in the invocation means "start a new version that succeeds the current top one". Decide MAJOR vs MINOR from the prompt's nature.

---

## 4. Filename + naming conventions

- **camelCase** action names for ingresses, handlers, outputs, emissions.
  - api: `createTenant.md`, `kcValidateCredentials.md`, `originateCall.md`
  - trigger: `subscriptionExpired.md`, `monthlyBillingTick.md`, `weatherChanged.md`, `callTryingTimeout.md`
  - consume: `cdrIngestKafka.md`, `partnerEventsRedis.md`
  - handler (optional): `tenantProvisioning.md`, `monthlyBillingCycle.md`
  - output: `tenant.md`, `dbcreated.md`, `recurringInvoice.md`, `cdrRow.md`
  - emission: `callTrying.md`, `callRinging.md`, `callAnswered.md`, `callHangup.md`
  - registry: exactly one `cause-codes.md` per version dir, sitting at the version root (flat sibling to `api/`, `trigger/`, `consume/`, `handler/`, `output/`, `emission/`).
- **Flat sibling categories at the version-dir root.** No `input/` wrapper. Every category is a top-level dir under `vX.Y/`.
- **Empty categories are kept as dirs.** A version that has no `trigger/`/`consume/`/`emission/` entries still gets the directory with a one-line `README.md` saying so. This makes the convention visible without forcing entries.
- **One file per API.** The synchronous response is a `## Response` section inside the api file, not a separate file. An API does one task — the doc stays small.
- **Long, self-descriptive names beat short cryptic ones.** A reader skimming the directory listing should infer the responsibility from the filename alone.
- Files inside each category sit flat (no sub-grouping) until there are >20 of one kind, then group by area.

---

## 5. File templates

### 5.1 api/<verb><Noun>.md  — inlined (default, no handler layer)

Use this template when the API's logic is self-contained — no other ingress runs the same flow. Steps live in the api file. This is the canonical default; only switch to the handler-style template (§5.1b) when sharing kicks in.

```markdown
# <verbNoun>()

HTTP API call. <one-sentence purpose>.

<optional 1–2 line context — security/auth, idempotency caveats, who calls this>.

## Params

- `<name>` (<Type>, required|optional) — <description>
- ...

## Computed (not supplied by caller)

- <field> — <how it's derived; only include if non-trivial>

## Steps

1. <step in order>
2. ...

## Produces

- [output/<name>](../output/<name>.md)
- ...

## Idempotency & retries

<How re-running the call behaves. Any deduping key. Whether earlier steps roll back on later-step failure.>

## Response

Universal envelope:

\`\`\`
{
  success:       bool,
  causeCode:     int | string,
  causeProtocol: string,        // "HTTP" | "<SERVICE>" | "SIP" | "SS7" | "GRPC" | ...
  description:   string,        // override_description from cause-codes.md if registered, else description
  payload:       <api-specific> // see Success below; usually null on failure
}
\`\`\`

REST: `causeCode` mirrors the HTTP status, `causeProtocol = "HTTP"`. gRPC/Java: `causeCode` from this service's custom registry — unless propagated from an upstream protocol (SIP/SS7/etc.), in which case `causeProtocol` is set to that family.

### Success

| causeCode | protocol | description (override if any) | payload                              |
|-----------|----------|-------------------------------|--------------------------------------|
| 200       | HTTP     | OK                            | [output/<name>](../output/<name>.md) |

### Failures

| causeCode | protocol | description (override if any)        | when it fires                        |
|-----------|----------|--------------------------------------|--------------------------------------|
| 400       | HTTP     | Bad Request                          | request body / query param malformed |
| ...       | ...      | ...                                  | ...                                  |

Every `(causeCode, protocol)` pair on this page must exist as a row in [../cause-codes.md](../cause-codes.md).

## Async events  (optional — only when the call streams events back to the caller)

- channel:     <ws / kafka topic / redis pubsub channel>
- correlation: <key, e.g. callId from the sync response>
- sequence:    <state-machine arrows: e.g. callTrying → (callRinging)* → callAnswered → callHangup>

  - [emission/<name>](../emission/<name>.md)
  - ...

## Notes

<Production caveats, security, dev-vs-prod differences.>
```

### 5.1b api/<verb><Noun>.md  — with shared handler

Use when 2+ ingresses run the same business flow. `## Steps`/`## Produces`/`## Idempotency` collapse to a single `## Fires handler` line — the handler file (§5.3b) carries them.

```markdown
# <verbNoun>()

<same as §5.1 down through ## Computed>

## Fires handler

- [handler/<name>](../handler/<name>.md)

## Response
<same as §5.1>
```

### 5.2 trigger/<name>.md  and  consume/<name>.md

Non-api ingresses: a domain event / timer / state-change (`trigger/`) or an async subscription (`consume/`). **No `## Response` section** — there's no sync caller.

```markdown
# <eventName>   ← e.g. subscriptionExpired (trigger), cdrIngestKafka (consume)

<One-sentence meaning. Describe how it fires (trigger) or what stream it subscribes to (consume).>

## Source                        ← trigger only
- cron `0 3 * * *`  | domain-event `SubscriptionExpired` | state-change <X→Y> | per-call timer armed at <transition>

## Subscription                  ← consume only
- broker:         kafka | redis | websocket | nats | ...
- coordinates:    topic `<name>` (key=...)  |  pubsub channel `<x>`  |  ws endpoint `<path>`
- consumer-group: `<group-id>`
- delivery:       at-least-once | exactly-once | at-most-once
- ordering:       per-key | unordered | session-ordered

## Payload

- `<field>` (<Type>) — <description>
- ...

## Steps                         ← omit and use `## Fires handler` instead when shared
1. ...

## Produces

- [output/<name>](../output/<name>.md)
- [emission/<name>](../emission/<name>.md)    ← if a trigger emits caller-facing events (e.g. callTryingTimeout → callHangup)

## Idempotency & retries / DLQ

<How re-runs / replays behave. Deduping key. For consume: retry policy, DLQ topic, replay safety.>

## Notes
```

### 5.3 output/<name>.md

```markdown
name: <name>
type: <Shape> (json | db row | kafka msg | file | log line | composite)
produced by: [<kind>/<name>](../<kind>/<name>.md)
              // <kind> = api | trigger | consume | handler
              // list multiple producers if more than one

<2–4 line description of the artifact, including which fields are notable, any state machine, any never-returned fields.>
```

Outputs are intentionally compact — name/type/producer lines are the contract; the prose is a reading aid.

### 5.3b handler/<name>.md  (optional layer)

Only present when 2+ ingresses share business logic, or when a single multi-step process is complex enough to deserve its own file.

```markdown
# <handlerName>

<One-sentence business meaning.>

## Activated by

- [api/<name>](../api/<name>.md)
- [trigger/<name>](../trigger/<name>.md)
- [consume/<name>](../consume/<name>.md)
- (or) chained from: [handler/<other>](./<other>.md)

## Steps

1. <step in order>
2. ...

## Produces

- [output/<name>](../output/<name>.md)
- ...

## Idempotency & retries

## Notes
```

### 5.3c emission/<name>.md  (caller-facing async stream)

Used by services with streaming APIs (e.g. EslCallStateMachine emits callTrying/callRinging/callAnswered/callHangup after the sync 202 response on `originateCall`).

```markdown
name: <name>
type: <Shape> (websocket frame (JSON) | kafka msg | redis pubsub msg | composite)
produced by: [<kind>/<name>](../<kind>/<name>.md)
              // most often `api/<x>` (after sync response) but `trigger/<x>` is valid too
              // (e.g. callTryingTimeout produces callHangup)
delivery:
  - websocket: <endpoint or eventChannel reference>
  - kafka:     topic `<name>`, key=<correlationKey>
  - redis:     pubsub channel `<pattern>`
correlation:       <key from the sync response, e.g. callId>
sequence-position: <first | non-terminal | terminal | terminal-on-failure>

## Cause codes carried

| causeCode | protocol | description (override if any) | when |
|----------:|----------|-------------------------------|------|
| ...       | ...      | ...                           | ...  |

All registered in [../cause-codes.md](../cause-codes.md).

## Payload shape

{ "type": "...", "callId": "...", "causeCode": ..., "causeProtocol": "...", "ts": ... }
```

### 5.4 cause-codes.md (one per version dir)

```markdown
# <Service> — Cause Codes (vX.Y)

The complete cause-code registry for this version. Every Response row references a row here.

## Reading the table

- **value** — wire value (int or string).
- **protocol** — `HTTP` | `<SERVICE_NAME>` (service-custom) | `SIP` | `SS7` | `GRPC` | ...
- **description** — standard / original meaning. For `HTTP`, the IANA reason phrase.
- **override_description** — service-specific phrasing returned to callers in the envelope. `—` means we send `description` verbatim.

## Codes

| value | protocol | description | override_description |
|------:|----------|-------------|----------------------|
| 200   | HTTP     | OK          | —                    |
| 400   | HTTP     | Bad Request | —                    |
| ...   | ...      | ...         | ...                  |
```

A few rules of thumb:
- Don't pre-populate codes you don't actually emit. Add rows lazily, when a Response file references them.
- `protocol` is mandatory — `404` alone is ambiguous (HTTP `Not Found` vs SIP `Not Found`).
- For service-custom codes, use a single all-caps protocol name = the service short name (e.g. `PARTY`, `ROUTESPHERE`). Numeric values for custom codes should not collide with HTTP standard codes — start from 1000+.

---

## 6. Cross-linking rules

- Every `api/<name>.md` includes a `## Response` section with at least one Success row and zero-or-more Failure rows.
- Every ingress file (api / trigger / consume) **either** has `## Steps` + `## Produces` inline **or** a `## Fires handler` line linking to a `handler/<name>.md`. Pick one, never both.
- Every handler file (when present) lists at least one **activation source** under `## Activated by` (ingress or chained handler) **and** at least one **output** (or emission) under `## Produces`.
- Every output / emission file's `produced by:` line names its producer — an ingress (default) or a handler (when used). Multiple producers are allowed; list them all.
- Every `(causeCode, protocol)` pair appearing in a `## Response` section or in an emission's "Cause codes carried" table must exist as a row in the same version's `cause-codes.md`.
- The `cause-codes.md` registry is internally complete — it never references another version's registry.
- Use **relative markdown links** with the `../` form, never absolute filesystem paths. With flat siblings, ingress → output is one `../` hop.
- Never link across versions. If a v1.0 file is identical in v1.1, copy it.

---

## 7. Workflow when the skill is invoked

Given `<version>` and `<prompt>`:

1. **Resolve the service.** Default = the service whose api-doc folder the user is currently working in (cwd hint, last-edited dir). If ambiguous, ask once.
2. **Resolve the version dir.**
   - `v1` / `latest` → highest existing dir matching pattern.
   - `v1.0`, etc. → exact, error if missing.
   - `next` → bump from current top per §3 rules; create dir by copying latest.
3. **Read the version's existing nodes.** List `input/api/*`, `trigger/*`, `output/*` so you understand what already exists. Avoid duplicate names; reuse existing triggers when the new behaviour fits.
4. **Decide which nodes the prompt needs.**
   - A new endpoint = one `api/<name>.md` (inlined steps; with `## Response` section) + one or more outputs. Add `## Async events` + emission files only if the call streams events back. **No handler unless reuse exists** — start inlined, promote later if needed.
   - A new internal/domain event = one `trigger/<name>.md` (no response).
   - A new async consumer = one `consume/<name>.md` (no response; declares broker, group, delivery, DLQ).
   - A new field on an existing ingress = edit the file (and bump if required-field). Update its `## Response` section if the new field affects success/failure modes.
   - **Promoting to a handler**: when a second ingress needs the same flow, lift the original ingress's `## Steps` / `## Produces` / `## Idempotency` into a new `handler/<name>.md`, then collapse those sections in **both** ingress files into `## Fires handler`. No version bump (no wire change).
5. **Write or edit files** following §5 templates. For any new `api/*`, the `## Response` section is part of the same file — never split.
6. **Update `cause-codes.md`.** Add any new rows the `## Response` section (or any emission's cause-codes-carried table) references. Don't duplicate rows. If a code already exists with a matching `(value, protocol)` and the override description still fits, just reference it.
7. **Wire cross-links** per §6.
8. **Decide if a version bump is required** per §3. If MAJOR, you should have used `next` — if not, surface the contradiction and pause.
9. **Surface unanswered questions** (security boundary, idempotency contract, who's the caller, which HTTP code best fits) — don't invent.

---

## 8. Worked examples

### 8a. Inlined (default — no handler, no streaming)

Invocation: `/apibuilder v1 "add a getKcClient endpoint"`

Service: `party`. Single new endpoint, self-contained logic, no reuse with anything else. Use the inlined template (§5.1).

```
v1.0/
├── api/
│   └── getKcClient.md          ← Params, Steps, Produces, Idempotency, Response  (all in one file)
├── output/
│   └── kcClient.md             ← produced by: api/getKcClient
└── cause-codes.md              ← add rows the Response references
```

No `handler/`, no `emission/`. MINOR bump.

### 8b. Promoted to handler (shared logic)

Invocation: `/apibuilder v1 "add an access control list for keycloak client"`

Two new apis (set + read) that **share** the same business logic for stamping/validating ACL rows. This earns a handler.

```
v1.0/
├── api/
│   ├── setKcClientAcl.md       ← POST. `## Fires handler` → kcClientAclManagement. Has `## Response`.
│   └── getKcClientAcl.md       ← GET.  `## Fires handler` → kcClientAclManagement. Has `## Response`.
├── handler/
│   └── kcClientAclManagement.md ← `## Activated by` lists both apis; carries Steps/Produces/Idempotency.
├── output/
│   ├── kcClientAcl.md          ← produced by: handler/kcClientAclManagement
│   └── aclStored.md            ← produced by: handler/kcClientAclManagement
└── cause-codes.md              ← add rows the apis' Response sections reference
```

Each file follows §5 templates. Cross-links wired per §6. Both apis reach the handler via `## Fires handler` — `## Steps` / `## Produces` / `## Idempotency` live in the handler only. MINOR bump.

### 8c. Promoting from inlined to handler

Already shipped: `setKcClientAcl.md` (inlined). Now adding `getKcClientAcl.md` that needs the same flow.

1. Lift `## Steps` + `## Produces` + `## Idempotency` from `setKcClientAcl.md` into a new `handler/kcClientAclManagement.md`.
2. Replace those three sections in `setKcClientAcl.md` with a single `## Fires handler` line.
3. Write the new `getKcClientAcl.md` with the same `## Fires handler` line.
4. Update `output/*.md` `produced by:` lines from the api to the handler.

No wire-contract change → **no version bump**. The promotion is a doc refactor.

### 8d. Streamed response (api + trigger + emission)

Invocation: `/apibuilder v1 "add originateCall on EslCallStateMachine"`

A streaming softswitch endpoint: the api accepts the call (sync `202`) then streams `callTrying / callRinging / callAnswered / callHangup` events back over a websocket / kafka / redis channel keyed by `callId`. A per-call timer aborts the call if it sits in TRYING too long.

```
v1.0/
├── cause-codes.md              ← rows for HTTP (202/4xx/5xx), SIP (100/180/200/486/487/503), ESLCALL (1001/1002)
├── api/
│   └── originateCall.md        ← Params, Steps, Produces, Idempotency,
│                                  ## Response (202 + 4xx/5xx),
│                                  ## Async events (lists the four emissions, channel, correlation, sequence)
├── trigger/
│   └── callTryingTimeout.md    ← Source (per-call timer at INIT→TRYING + N seconds), Payload, Steps,
│                                  Produces (cdrRow + callHangup emission)
├── emission/
│   ├── callTrying.md           ← non-terminal; SIP 100 row
│   ├── callRinging.md          ← non-terminal; SIP 180 row
│   ├── callAnswered.md         ← non-terminal; SIP 200 row
│   └── callHangup.md           ← terminal; produced by api/originateCall AND trigger/callTryingTimeout
│                                  carries SIP 200/486/487/503 (pass-through) + ESLCALL 1001 (timeout, override)
└── output/
    └── cdrRow.md               ← produced by: api/originateCall, trigger/callTryingTimeout
```

Notable cross-links:
- `originateCall.md` `## Async events` lists all four `emission/*.md` files.
- `callHangup.md` has two `produced by:` entries.
- `callHangup.md`'s "cause codes carried" table mixes pass-through `SIP` codes (no override) and the custom `ESLCALL 1001` (with override description "Trying timeout — call sat in TRYING longer than N s").
- The trigger doc has no `## Response` — it's an internal event, not a sync caller.

This is the canonical streaming/timeout shape. MINOR bump (all additive).

---

## 9. When not to use this skill

- **Per-endpoint reference docs.** Those live under `<service>/todo/` (the parked OpenAPI-ish style) and are out of scope here. The skill operates only on the input/trigger/output graph.
- **Code generation.** This skill writes documentation, not Java. Code work needs a different skill.
- **Cross-service contracts** that don't fit one service's input/trigger/output. Those go in `architecture/docs/<topic>/` directly.

---

## 10. Reference materials

- **Live canonical example** — `/home/mustafa/telcobright-projects/architecture/docs/api-doc/party/v1.0/`. Mirror its file shapes exactly.
- **Service-folder README** — every `<service>/README.md` should explain versioning + the three-layer model in 30 lines, copy-paste from the party README and adjust service-specific bits.
- **The reading rule** — when a node file gets longer than ~50 lines, you've drifted into reference docs; split or trim. Keep nodes brief; the directory structure carries the model.
