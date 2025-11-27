# Ollama Adapter — haa

## References

- `/README-DEV.md` — Development setup, commands, validation pipeline
- `/.llm-agent/refs/` — Generated context (e.g., `hometopology.md`)
- `/.llm-agent/prompts/` — Reusable task prompts

## Initialization

1. Run `make setup`
2. Read `/README-DEV.md`
3. Read any files in `/.llm-agent/refs/`
4. Excecute `make pull (assist with setup eg ssh if that fails)`
5. Execute `/.llm-agent/prompts/hometopology-prompt.md` to generate home topology

## Workflow

1. Prime model with system prompt below
2. Describe change; model drafts YAML/Python
3. Apply patch manually
4. To support memory management, store notes for llm or human use in `/.llm-agent/refs/` as appropriate
5. Run `make validate`
6. Deploy with `make push` after validation passes

## Commands

| Command                                  | Purpose                      |
| ---------------------------------------- | ---------------------------- |
| `make pull`                            | Sync config from HA          |
| `make validate`                        | YAML + entity + HA checks    |
| `make push`                            | Validate → deploy → reload |
| `make entities ARGS='--search motion'` | Query entity registry        |

## Guardrails

- Local models may hallucinate entity IDs — confirm via `make entities`
- Always validate before push
- Set `USE_RSYNC_SUDO=1` in `.env` if HA host requires sudo for rsync

## System Prompt

```
Home Assistant automation engineer using haa toolkit.
Plan changes first, provide diffs, ask user to run `make validate`.
Never request `make push` until validation output is clean.
Query entities before referencing IDs in YAML.
```
