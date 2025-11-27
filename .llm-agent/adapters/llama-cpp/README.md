# llama.cpp Adapter — haa

## References
- `/README-DEV.md` — Development setup, commands, validation pipeline
- `/.llm-agent/refs/` — Generated context (e.g., `hometopology.md`)
- `/.llm-agent/prompts/` — Reusable task prompts

## Initialization

1. Run `make setup`
2. Read `/README-DEV.md`
3. Read any files in `/.llm-agent/refs/`
4. Execute `/.llm-agent/prompts/hometopology-prompt.md` to generate home topology

## Workflow

1. Prime model with system prompt below
2. Request a plan before edits
3. Apply patches manually
4. Run `make validate`
5. Deploy with `make push` after validation passes

## Commands

| Command | Purpose |
|---------|---------|
| `make pull` | Sync config from HA |
| `make validate` | YAML + entity + HA checks |
| `make push` | Validate → deploy → reload |
| `make entities ARGS='--area kitchen'` | Area-specific entity view |

## Guardrails

- Quote entity IDs from `make entities` output before editing YAML
- `.llm-agent/hooks/` auto-runs YAML formatting and validation
- Set `USE_RSYNC_SUDO=1` in `.env` if HA host requires sudo for rsync

## System Prompt

```
Home Assistant automation engineer using haa toolkit.
Plan changes first, reference README-DEV.md for architecture.
Provide diffs, ask user to run `make validate`.
Never request `make push` until validation is clean.
```
