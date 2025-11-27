# haa Development Guide

LLM-assisted development toolkit for Home Assistant automations. Works with Claude Code, GitHub Copilot, Ollama, llama.cpp, and other assistants.

## Repository Layout

| Path | Purpose |
|------|---------|
| `config/` | HA configuration mirrored from your instance |
| `tools/` | Python utilities: validation, entity discovery, deployment |
| `.llm-agent/` | Hooks, adapters, prompts, and generated references |
| `.llm-agent/adapters/` | Per-assistant configs (claude-code, vscode-copilot, ollama, llama-cpp) |
| `.llm-agent/refs/` | Generated context files (e.g., `hometopology.md`) |
| `.llm-agent/prompts/` | Reusable prompts for common tasks |
| `Makefile` | Primary CLI: pull, push, validate, entities |

## Quick Start

```bash
make setup          # Install Python deps + pre-commit hooks
make pull           # Sync config from HA instance
make validate       # YAML + entity + HA validation
make push           # Validate → rsync → reload HA
make entities ARGS='--search motion'  # Query entity registry
```

## Validation Pipeline

1. **Sync** — `make pull` mirrors `/config` from HA via rsync
2. **Edit** — LLM proposes YAML/script changes
3. **Validate** — `make validate` runs:
   - YAML syntax (HA-specific tags: `!include`, `!secret`, `!input`)
   - Entity/device reference checks against `.storage/` data
   - Official HA config validator
4. **Deploy** — `make push` re-validates, rsyncs, triggers HA reload

`.llm-agent/hooks/` enforces guardrails automatically:
- `pretooluse-ha-push-validation.sh` — blocks push on validation failure
- `posttooluse-ha-validation.sh` — post-edit YAML checks
- `yaml-formatter.sh` — canonical YAML formatting

## Entity Naming Convention

Format: `location_room_device_sensor`

```
binary_sensor.home_basement_motion_battery
media_player.office_main_bedroom_sonos
climate.cabin_living_room_heatpump
```

- **Location**: `home`, `office`, `cabin`
- **Room**: `kitchen`, `living_room`, `garage`
- **Device**: Hardware identifier (`heatpump`, `sonos`, `aqara_hub`)
- **Sensor**: Measurement/capability (`battery`, `status`)

Ask clarifying questions when entity naming is ambiguous.

## Development Tools

### Code Quality
- **Black** — PEP8 formatting (line length: 88)
- **isort** — Import sorting (black-compatible)
- **flake8** — Style enforcement + plugins
- **pylint** — Code analysis
- **mypy** — Static type checking

### Testing
- **pytest** + **pytest-cov** — Tests with 80% coverage target
- **pytest-mock** — Mocking utilities

### Automation
- **pre-commit** — Git hooks for all checks
- **yamllint** — HA-compatible YAML validation

## Dev Commands (Makefile.dev)

```bash
make -f Makefile.dev dev-setup        # Full dev environment setup
make -f Makefile.dev dev-format       # Auto-format code
make -f Makefile.dev dev-check-all    # Run all quality checks
make -f Makefile.dev dev-test-coverage # Tests with coverage report
make -f Makefile.dev dev-workflow     # Format + lint + test
make -f Makefile.dev dev-pre-commit   # Install git hooks
```

## Configuration

| Tool | Setting |
|------|---------|
| Black | Line length 88, Python 3.11+ |
| isort | Profile: black |
| flake8 | Ignores E203, W503, E501 (Black compat) |
| mypy | Strict mode, ignores missing third-party imports |
| pytest | 80% coverage target, HTML reports |

## Troubleshooting

```bash
# Recreate venv symlink
rm venv && ln -sf ../venv venv

# Update deps
make -f Makefile.dev dev-update-deps

# Skip hooks temporarily
git commit --no-verify -m "message"
```
