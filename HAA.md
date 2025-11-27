# Home Assistant Assistant (haa)

`haa` is a generalized devkit for building, validating, and deploying Home Assistant automations with help from any LLM-powered assistant. This document captures the repository architecture plus the adapter contract that lets Claude Code, GitHub Copilot, Ollama, llama.cpp, and future tooling plug into the same workflow.

## Core Layout

| Path | Purpose |
| --- | --- |
| `config/` | Real Home Assistant configuration mirrored from your instance |
| `tools/` | Python utilities for validation, entity discovery, and deployment |
| `.llm-agent/` | Shared automation hooks, guard rails, and adapter metadata |
| `adapters/<id>/` | Documentation and scaffolding for each LLM/environment |
| `Makefile` | High-level pull, push, validation, and formatting commands |
| `Taskfile.yml` *(planned)* | Cross-platform task runner once `uv` adoption lands |

Shipped adapters live under `adapters/`:

- `claude-code`
- `vscode-copilot`
- `ollama`
- `llama-cpp`

All automation helpers (pre-push validation, YAML formatting, Python quality checks) now live under `.llm-agent/`. Nothing inside these hooks is agent-specific; adapters simply describe how to expose repo commands to a specific assistant.

## Adapter Contract

Each adapter directory carries an `adapter.json` manifest plus optional helper assets:

```json
{
  "id": "vscode-copilot",
  "name": "GitHub Copilot in VS Code",
  "editor": "vscode",
  "entrypoints": [
    {"command": "make push", "description": "Validate + deploy config"},
    {"command": "make entities ARGS='--search fridge'", "description": "Query entity registry"}
  ],
  "docs": ["README.md"],
  "env": {
    "requirements": ["python>=3.11", "uv"],
    "recommendedExtensions": ["github.copilot-chat"]
  }
}
```

Adapters may also include:

- `README.md` explaining workflow expectations for that assistant
- `prompts/` with starter prompt snippets
- `tasks/` or `snippets/` wrapping frequently used CLI invocations

## Validation + Deployment Lifecycle

1. **Sync** — `make pull` mirrors `/config` from Home Assistant via rsync.
2. **Plan** — The adapter guides the LLM to propose YAML edits or script updates.
3. **Validate locally** — `make validate` (calls `tools/run_tests.py`) runs:
   - YAML syntax checks with HA-specific tags (`!include`, `!secret`, `!input`).
   - Entity & device reference validation backed by `.storage/` data.
   - The official Home Assistant validator for integration-level issues.
4. **Push** — `make push` re-runs validation, rsyncs back to HA, then triggers the reload API.
5. **Observe** — Entity explorer + log helpers surface runtime context for next iterations.

`.llm-agent/hooks/` enforces safety rails:

- `pretooluse-ha-push-validation.sh` blocks deployments when validation fails.
- `posttooluse-ha-validation.sh` runs quick checks after YAML edits.
- `posttooluse-python-quality.sh` consolidates formatters/linters (moving to `ruff`).
- `yaml-formatter.sh` enforces canonical YAML structure.

## Entity Naming Convention

haa projects use the shared `location_room_device_sensor` format:

```
binary_sensor.home_basement_motion_battery
media_player.office_main_bedroom_sonos
climate.cabin_living_room_heatpump
```

Guidelines:

- **Location**: `home`, `office`, `cabin`, etc.
- **Room**: `kitchen`, `living_room`, `garage`, etc.
- **Device**: Canonical hardware identifier (`heatpump`, `sonos`, `aqara_hub`).
- **Sensor**: Specific measurement or capability (`battery`, `status`).

Assistants must ask clarifying questions when ambiguity exists before changing YAML.

## Using the Toolkit with an LLM

1. Stand up tooling via `make setup` (migrating to `uv sync`).
2. Connect your assistant using the relevant adapter instructions.
3. Describe the change at a high level ("Add an automation to turn off garden lights at dawn").
4. Review generated YAML, run `make validate`, then stage changes for review.
5. Deploy with `make push` once satisfied.

## Roadmap Snapshot

- **uv adoption** for reproducible, fast Python environments.
- **ruff** to replace Black + isort + flake8.
- **Taskfile** to align CLI UX across shells.
- **Additional adapters** (Cursor, Cody, Continue) tracked in `ForkPlan.md`.

Questions or adapter ideas? Open an issue referencing this document so we can extend the manifest schema without breaking existing tooling.
