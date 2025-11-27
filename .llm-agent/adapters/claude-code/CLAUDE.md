# Claude Code Adapter

Use Anthropic's Claude Code desktop app to collaborate on Home Assistant automations powered by the `haa` toolkit.

## Prerequisites

- Claude Code (latest desktop build) with file-system access to this repo
- SSH access from your workstation to the Home Assistant host
- Python 3.11+ plus the `haa` virtual environment (`make setup`)
- Optional: [Advanced SSH &amp; Web Terminal](https://github.com/hassio-addons/addon-ssh) add-on inside Home Assistant

## Recommended Workflow

1. **Open the repository in Claude Code** and pin `.llm-agent/settings.json` so Claude loads the guard-rails.
2. **Describe the automation** you want ("add a dawn lights-off automation").
3. **Let Claude propose edits**; validate YAML output using `make validate` (Claude knows the command from the adapter manifest).
4. **Review diffs locally**. Claude can open files but you should confirm changes with `git diff`.
5. **Deploy with safeguards** using `make push` once validation passes.

## Helpful Commands

Claude should prioritize the following commands (also listed in `adapter.json`):

- `make pull` — sync your live HA config before editing
- `make validate` — run YAML/entity/offical HA checks locally
- `make push` — re-run validation, deploy via rsync, and reload HA
- `make entities ARGS='--search fridge'` — inspect entity registry slices

## Prompt Snippetsgit add -A

```
You are operating inside the haa repo. Plan YAML changes first, then run `make validate`. Never call `make push` until validation passes.
```

```
When you need entity IDs, run `make entities ARGS='--search <term>'` and summarize the results before editing YAML.
```

## Tips

- The `.llm-agent/` hooks automatically format YAML and block unsafe pushes.
- Claude can edit multiple files at once; ask it to keep automations small and focused.
- Mention `USE_RSYNC_SUDO=1` in `.env` if your HA user needs sudo privileges for rsync.
