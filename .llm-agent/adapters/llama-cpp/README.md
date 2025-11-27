# llama.cpp Adapter

This adapter documents how to pair `haa` with self-hosted models exposed via [llama.cpp](https://github.com/ggerganov/llama.cpp) (CLI or server mode).

## Prerequisites

- A recent llama.cpp build (server or `llama.cpp --interactive-first` shell)
- Downloaded model (e.g., `llama-3.1-70b-instruct.Q4_K_M.gguf`)
- Shell or UI that forwards your prompts to llama.cpp
- Local checkout of the haa repository with SSH access to your Home Assistant host
- Python 3.11+ tools installed via `make setup`

## Usage Pattern

1. **Prime the model** with a system prompt describing the haa workflow (see `prompts/llama-cpp.txt`, add your own variants).
2. **Request a plan** before edits. Ask the model to reference `HAA.md` for architecture rules.
3. **Apply patches manually** (recommended) or via automation.
4. **Validate** with `make validate`.
5. **Deploy** using `make push` only after validation is green.

## Common Commands

- `make pull`
- `make validate`
- `make push`
- `make entities ARGS='--area kitchen'`

## Prompt Tips

- Remind the model that `.llm-agent/hooks/` auto-runs YAML formatting and validation.
- Ask it to quote entity IDs from `make entities` output before editing YAML.
- Mention the `USE_RSYNC_SUDO` flag if your HA host requires sudo for rsync.

## Extending

Add any llama.cpp-specific helper scripts, prompt templates, or workflows inside this directory so they remain isolated from upstream files.
