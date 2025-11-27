# VS Code Copilot Adapter — haa

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

1. Open folder in VS Code with Copilot Chat enabled
2. Generate/modify automations; cite entity IDs
3. Run validation in integrated terminal
4. To support memory management, store notes for llm or human use in `/.llm-agent/refs/` as appropriate
5. Review diffs before deployment
6. Deploy with `make push` after validation passes

## Commands

| Command                                   | Purpose                      |
| ----------------------------------------- | ---------------------------- |
| `make pull`                             | Sync config from HA          |
| `make validate`                         | YAML + entity + HA checks    |
| `make push`                             | Validate → deploy → reload |
| `make entities ARGS='--domain climate'` | Domain-specific entity view  |

## Guardrails

- `.llm-agent/hooks/` auto-runs YAML formatting and blocks unsafe pushes
- Always validate before push
- Set `USE_RSYNC_SUDO=1` in `.env` if HA host requires sudo for rsync

## System Prompt

```
Assisting with haa repo. Plan changes, show files to touch, run `make validate` before `make push`.
Query entities before editing YAML. Remind about USE_RSYNC_SUDO if SSH writes need sudo.
```
