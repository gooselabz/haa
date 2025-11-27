# VS Code + GitHub Copilot Adapter

These instructions help GitHub Copilot (Chat or inline) operate safely inside the `haa` repository.

## Prerequisites

- VS Code 1.95+ with:
  - [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
  - [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat)
- Terminal access to the repo (WSL, macOS, or Linux)
- Python 3.11+ and `make setup` completed
- SSH access from your machine to the Home Assistant host

## Workflow

1. **Open the folder in VS Code** and enable Copilot Chat for the workspace.
2. **Reference this adapter config**: point Copilot to `adapters/vscode-copilot/adapter.json` for hooks and validation rules.
3. **Generate or modify automations**. Ask Copilot to explain reasoning and cite entity IDs.
4. **Run validation commands in the integrated terminal** (Copilot can paste them, you hit enter).
5. **Inspect diffs (`git status`, `git diff`)** yourself before deployment.
6. **Deploy** via `make push` once validation is green.

## Common Commands

- `make pull` — refresh local config from Home Assistant
- `make validate` — fast signal on YAML + entity references
- `make push` — rsync to HA with validation guard
- `make entities ARGS='--domain climate'` — domain-specific entity view

## Copilot Prompt Template

```
You are assisting with the Home Assistant Assistant (haa) repo.
Always plan changes, show the files you will touch, and run `make validate` before suggesting `make push`.
If SSH writes require sudo on the HA host, remind me to set USE_RSYNC_SUDO=1 in .env.
```

## Notes

- `.llm-agent/hooks/` runs YAML formatting and validation automatically (configured in `adapter.json`).
- Use CodeLens or Copilot Chat to summarize test failures before editing.
- Keep adapter-specific documentation inside `adapters/vscode-copilot/` when you extend this workflow.
