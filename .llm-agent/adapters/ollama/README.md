# Ollama Adapter

Run haa workflows with local models served by [Ollama](https://ollama.ai) (e.g., `llama3`, `phi3`, `mistral`).

## Prerequisites

- Ollama installed with at least one code-capable model
- A terminal/chat interface wired into Ollama (e.g., Continue.dev, Open WebUI, custom scripts)
- Local checkout of the haa repository with SSH access to your Home Assistant host
- Python 3.11+ virtual environment (`make setup`)

## Suggested Workflow

1. **Prime your model** with the system prompt from `prompts/ollama.txt` (create more as needed).
2. **Describe the change** you want. Have the model draft YAML or Python updates.
3. **Apply the patch manually** (or with tooling like `apply_patch`).
4. **Validate** using `make validate`.
5. **Deploy** with `make push` once validation succeeds.

## Key Commands

- `make pull`
- `make validate`
- `make push`
- `make entities ARGS='--search motion'`

## Prompt Starter

```
You are a Home Assistant automation engineer using the haa toolkit.
Plan changes first, provide diffs, then ask me to run `make validate`.
Never request `make push` until validation output is clean.
```

## Notes

- Local models may hallucinate entity IDs—always confirm via `make entities`.
- Use `USE_RSYNC_SUDO=1` in `.env` if your HA host requires sudo for rsync.
- Store any additional prompt snippets inside this directory to keep adapters isolated.
