# tb-architecture

Cross-service architecture documentation for the telcobright stack: API contracts, ecosystem maps, and AI skills.

## Layout

```
docs/
├── api-doc/         service-by-service contract surfaces (input/trigger/output model)
│   └── <service>/
│       ├── README.md
│       ├── vMAJOR.MINOR/    versioned snapshots (input/, trigger/, output/)
│       └── todo/            parked detailed reference docs
├── ai-skills/       reusable AI-agent skills referencing the conventions in this repo
│   └── apibuilder.md
├── ecosystem/       per-component / per-integration design notes
└── orchestrix-odoo/ orchestrix-on-Odoo subsystem docs
```

## Conventions

- API documentation follows the **versioned input/trigger/output** model — see [docs/api-doc/party/README.md](docs/api-doc/party/README.md) for the canonical example and [docs/ai-skills/apibuilder.md](docs/ai-skills/apibuilder.md) for the full authoring contract.
- Each service gets one folder under `docs/api-doc/<service>/`. Inside it, each `vX.Y/` folder is a self-contained snapshot of the contract surface; old versions stay frozen so callers can pin against them.
- AI skills live in `docs/ai-skills/<name>.md`. The skill file is self-contained — point an agent at the path and they'll have everything they need.
