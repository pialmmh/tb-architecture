# Party — Versioned Input / Trigger / Output

> **Adding or changing API docs in this folder?** Use the **apibuilder** skill — it has the full convention, version rules, file templates, and worked examples.
>
> - Skill file: [`/home/mustafa/telcobright-projects/architecture/docs/ai-skills/apibuilder.md`](../../ai-skills/apibuilder.md)
> - Slash invocation (where supported): `/apibuilder <version> <prompt>` — e.g. `/apibuilder v1 "add an access control list for keycloak client"`
> - Or just hand the skill file path to another agent and they'll pick up the conventions cold.
>
> The summary below is enough to navigate; reach for the skill file when authoring.

---

Each `vX.Y/` folder is a self-contained snapshot of the contract surface at that version.

```
party/
├── README.md                 ← you are here
├── v1.0/                     ← current
│   ├── input/                what the service receives (api call, kafka msg, clock tick, config load, …)
│   │   └── api/
│   ├── trigger/              named business event/action; fires when an input arrives,
│   │                         OR on a clock, OR as a chain reaction from another trigger
│   └── output/               named, typed artifact produced by a trigger
│                             (json, db row, kafka msg, file, log)
└── todo/                     deferred per-endpoint API docs parked here until later
```

**Current version:** `v1.0`. Live nodes:

| Layer | File | Notes |
|---|---|---|
| input | [v1.0/input/api/createTenant.md](v1.0/input/api/createTenant.md) | operator implicit (YAML-bound); appName=orchestrix only |
| input | [v1.0/input/api/kcFindUserByUsername.md](v1.0/input/api/kcFindUserByUsername.md) | LAN-only SPI |
| input | [v1.0/input/api/kcValidateCredentials.md](v1.0/input/api/kcValidateCredentials.md) | LAN-only SPI |
| trigger | [v1.0/trigger/tenantProvisioning.md](v1.0/trigger/tenantProvisioning.md) | activated by createTenant() |
| trigger | [v1.0/trigger/provideAuthToKeyCloak.md](v1.0/trigger/provideAuthToKeyCloak.md) | activated by the two SPI inputs; read-only |
| output | [v1.0/output/tenant.md](v1.0/output/tenant.md) | json + master DB row |
| output | [v1.0/output/dbcreated.md](v1.0/output/dbcreated.md) | one or many per provision run |
| output | [v1.0/output/kcUserView.md](v1.0/output/kcUserView.md) | Keycloak-shaped user record |
| output | [v1.0/output/kcCredentialsValidation.md](v1.0/output/kcCredentialsValidation.md) | `{valid: bool}` |

## How the three relate

```
input  ─fires→  trigger  ─produces→  output
                   ↑                    ↑
              (also fired by         (output may have
               clock or another       multiple triggers)
               trigger)
```

- An **input** alone never produces an output directly — it fires a trigger, the trigger does the work. This is what lets us model behaviour like "a monthly recurring invoice is generated when a subscription is triggered": the subscription input arms a recurring trigger; the trigger (not the original input) produces the invoice output each cycle.
- A **trigger** is the unit of business behaviour. It has activation sources (one or more) and produces artifacts (one or more).
- An **output** records its trigger(s) — the audit answer to "what caused this artifact to exist?".

## Per-file conventions (summary)

- **input/** files: what the call looks like + which trigger(s) it fires.
- **trigger/** files: activation sources, what it does, what it produces, idempotency + retries.
- **output/** files: `name`, `type`, `trigger(s)`, brief description.
- camelCase action names; long + self-descriptive over short + cryptic.
- **Cross-link within the same version only.** Don't link `v1.1/` → `v1.0/`; copy the file if it's reused.

Full templates with section-by-section structure live in the skill file (see top of this README).

## Versioning rules (summary)

| Change | Bump |
|---|---|
| New input / trigger / output | MINOR |
| New optional field on an existing one | MINOR |
| Removed input / renamed trigger / changed output shape | MAJOR |
| Added required field to an existing input | MAJOR |
| Typo, prose clarification | no bump |

- Bumping = **copy the latest version dir to a new `vX.Y/` and edit there**. Don't delta-edit; each version must be self-contained.
- A version dir **freezes once a release using it ships**. Edits after that are limited to clarifications that don't change the wire contract.
- The "current" version is whichever directory name is highest. No symlinks. If parallel versions are intentionally live, add a one-line `current: v1.0` here.

## Where the deferred reference docs went

The first cut of this folder was 56 per-endpoint reference docs (one file per HTTP method/path) plus an `inputs-and-outputs.md` map. Those are parked under [`todo/`](todo/) — useful as a reference, not the live design surface. As triggers come online they'll get distilled back into the input/trigger/output graph; until then `todo/` stays frozen.
