# Party — Versioned Service Contract

> **Adding or changing API docs in this folder?** Use the **apibuilder** skill — it has the full convention, version rules, file templates, and worked examples.
>
> - Skill file: [`/home/mustafa/telcobright-projects/architecture/docs/ai-skills/apibuilder.md`](../../ai-skills/apibuilder.md)
> - Slash invocation (where supported): `/apibuilder <version> <prompt>` — e.g. `/apibuilder v1 "add an access control list for keycloak client"`
> - Or just hand the skill file path to another agent and they'll pick up the conventions cold.
>
> The summary below is enough to navigate; reach for the skill file when authoring.

---

Each `vX.Y/` folder is a self-contained snapshot of the contract surface at that version. The version dir uses **flat sibling categories** — no `input/` wrapper.

```
party/
├── README.md                 ← you are here
├── v1.0/                     ← current
│   ├── cause-codes.md        per-version cause-code registry
│   ├── api/                  sync ingress (REST/gRPC/Java) — has `## Response` (+ optional `## Async events`)
│   ├── trigger/              internal events — timer, domain event, state change   (empty in v1.0)
│   ├── consume/              async subscribers — kafka / redis / ws server          (empty in v1.0)
│   ├── emission/             caller-facing async streams — ws / kafka / redis       (empty in v1.0)
│   └── output/               artifacts — db row, internal kafka msg, file, log
└── todo/                     deferred per-endpoint reference docs (frozen pre-design surface)
```

`handler/` is an optional sibling layer (not present in v1.0 — see "When the optional `handler/` layer would be added" below).

**Current version:** `v1.0`. Live nodes:

| Layer | File | Notes |
|---|---|---|
| api | [v1.0/api/createTenant.md](v1.0/api/createTenant.md) | operator implicit (YAML-bound); appName=orchestrix only — response: 201 / 400 / 409 / 422 / 500 / 503 |
| api | [v1.0/api/kcFindUserByUsername.md](v1.0/api/kcFindUserByUsername.md) | LAN-only SPI — response: 200 / 400 / 401 / 404 / 500 / 503 |
| api | [v1.0/api/kcValidateCredentials.md](v1.0/api/kcValidateCredentials.md) | LAN-only SPI — response: 200 (`{valid: bool}`, `false` still 200) / 400 / 401 / 500 / 503 |
| trigger | (none) | placeholder dir + README |
| consume | (none) | placeholder dir + README |
| emission | (none) | placeholder dir + README |
| output | [v1.0/output/tenant.md](v1.0/output/tenant.md) | json + master DB row |
| output | [v1.0/output/dbcreated.md](v1.0/output/dbcreated.md) | one or many per `createTenant()` |
| output | [v1.0/output/kcUserView.md](v1.0/output/kcUserView.md) | Keycloak-shaped user record |
| output | [v1.0/output/kcCredentialsValidation.md](v1.0/output/kcCredentialsValidation.md) | `{valid: bool}` |
| registry | [v1.0/cause-codes.md](v1.0/cause-codes.md) | single source of truth for cause codes in v1.0 |

## How the layers relate

```
ingress  (one of api / trigger / consume)  ─produces→  output      (artifacts: db row, kafka msg, file)
   │                                                                — what was written
   ├─returns──→  ## Response section            (api only — sync envelope to the caller)
   │
   └─emits────→  emission                       (api only — caller-facing async stream)
```

- An **api** file carries everything needed to implement and call the endpoint: `Params`, `Steps` (what the call does, in order), `Produces` (artifacts written), `Idempotency`, and `## Response` (sync envelope + cause-code rows). Optionally `## Async events` listing emissions.
- A **trigger** file describes an internal stimulus (cron, domain event, state change), its payload, what it does, and what it produces. No `## Response` — there's no synchronous caller.
- A **consume** file describes an async subscription (kafka / redis / ws inbound), declaring broker, consumer-group, delivery guarantee, DLQ, and replay policy.
- An **output** records its producer — bidirectional: the api/trigger/consume file lists outputs under `## Produces`; the output's `produced by:` line names the producer.
- An **emission** is a caller-facing async stream (websocket frame / kafka per-call topic / redis pubsub) keyed by a correlation id from the sync response. Listed in the api file's `## Async events` section.
- The **`## Response` section** lives inside each api file (one file per API; an API does one task, the doc stays small). It carries the universal envelope (`success`, `causeCode`, `causeProtocol`, `description`, `payload`) and tabulates every cause code the call can return. Every `(causeCode, protocol)` pair must exist as a row in the version's [cause-codes.md](v1.0/cause-codes.md).
- The **cause-code registry** (`cause-codes.md`) is the single source of truth for every code the service emits at this version. Adding a row is MINOR; removing or repurposing a row is MAJOR.

### When the optional `handler/` layer would be added

Skip until at least one of these is true:
- A multi-step process is activated by **more than one** ingress (e.g. `tenantReprovision()` invoked from both an api call and a periodic timer — the steps belong in `handler/` so they aren't duplicated).
- An async/internal stimulus needs to run the same flow as a sync api.
- The `## Steps` section of a single ingress grows past ~10 steps or has reusable sub-flows worth their own files.

None of these apply to party today. If they do later, copy the ingress's `## Steps` / `## Produces` / `## Idempotency` sections into a new `handler/<name>.md` and replace them in the ingress(es) with `## Fires handler` linking to it.

### The hard rule

Every API in this service returns a value — REST, gRPC, or internal Java. There is no fire-and-forget endpoint. REST uses standard HTTP status codes; gRPC/Java uses the service's custom registry. Upstream protocol codes (SIP, SS7) propagate verbatim with `causeProtocol` set to the upstream family. See the [apibuilder skill](../../ai-skills/apibuilder.md) §0 for the envelope and per-protocol expectations.

## Per-file conventions (summary)

- **api/** files: `Params`, `Computed` (if any), `Steps`, `Produces` (links to outputs), `Idempotency`, `## Response` (success rows + failure rows + envelope shape), optional `## Async events`, `Notes`. One file per API — never split.
- **trigger/, consume/** files: same shape minus `## Response` and `## Async events`. Trigger declares its `Source` (cron / domain-event / state-change). Consume declares broker, consumer-group, delivery guarantee, DLQ, replay policy.
- **output/** files: `name`, `type`, `produced by:`, brief description. The `produced by:` line links to the ingress (api/trigger/consume) — or, when the optional `handler/` layer is in use, to the handler.
- **emission/** files: `name`, `type`, `produced by:`, `delivery:` (channel list), `correlation:` (the key tying it to a sync response), `sequence-position:`, plus a cause-codes-carried table.
- **`cause-codes.md`** at the version-dir root: 4 columns — `value`, `protocol`, `description`, `override_description`.
- camelCase action names; long + self-descriptive over short + cryptic.
- **Cross-link within the same version only.** Don't link `v1.1/` → `v1.0/`; copy the file if it's reused. Same applies to `cause-codes.md` — copies on bump.

Full templates with section-by-section structure live in the skill file (see top of this README).

## Versioning rules (summary)

| Change | Bump |
|---|---|
| New api / trigger / consume / output / emission | MINOR |
| New optional field on an existing ingress | MINOR |
| New cause-code row in `cause-codes.md` | MINOR |
| New cause-code row added to an api's `## Response` table | MINOR |
| Removed ingress / changed output shape | MAJOR |
| Added required field to an existing ingress | MAJOR |
| Removed cause-code row, or changed its `value`/`protocol` | MAJOR |
| Removed a cause-code row from an api's `## Response` (caller-visible) | MAJOR |
| Edited `description` / `override_description` text | no bump |
| Edited `## Steps` text without changing the wire contract | no bump |
| Promoting inlined steps to a `handler/` file (no wire change) | no bump |
| Typo, prose clarification | no bump |

- Bumping = **copy the latest version dir to a new `vX.Y/` and edit there**. Don't delta-edit; each version must be self-contained.
- A version dir **freezes once a release using it ships**. Edits after that are limited to clarifications that don't change the wire contract.
- The "current" version is whichever directory name is highest. No symlinks. If parallel versions are intentionally live, add a one-line `current: v1.0` here.

## Where the deferred reference docs went

The first cut of this folder was 56 per-endpoint reference docs (one file per HTTP method/path) plus an `inputs-and-outputs.md` map. Those are parked under [`todo/`](todo/) — useful as a reference, not the live design surface. As ingress points come online they'll get distilled back into the typed graph; until then `todo/` stays frozen.
