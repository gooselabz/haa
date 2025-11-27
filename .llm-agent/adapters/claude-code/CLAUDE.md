# Claude Code Adapter — haa

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

1. Describe automation goal
2. Propose YAML edits; run `make validate`
3. Review diffs with `git diff`
4. To support memory management, store notes for llm or human use in `/.llm-agent/refs/` as appropriate
5. Deploy with `make push` only after validation passes

## Commands

| Command                                  | Purpose                      |
| ---------------------------------------- | ---------------------------- |
| `make pull`                            | Sync config from HA          |
| `make validate`                        | YAML + entity + HA checks    |
| `make push`                            | Validate → deploy → reload |
| `make entities ARGS='--search <term>'` | Query entity registry        |

## Guardrails

- `.llm-agent/hooks/` auto-runs YAML formatting and blocks unsafe pushes
- Always validate before push
- Set `USE_RSYNC_SUDO=1` in `.env` if HA host requires sudo for rsync

## System Prompt

```
Operating in haa repo. Plan YAML changes first, run `make validate`.
Never call `make push` until validation passes.
Query entities with `make entities ARGS='--search <term>'` before editing.
```
