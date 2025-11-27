# Home Assistant Assistant (haa)

An LLM-agnostic devkit for Home Assistant automations and configuration management.

This repository is a forward-looking fork of [philippb/claude-homeassistant](https://github.com/philippb/claude-homeassistant), generalized so multiple LLM assistants can collaborate on Home Assistant automations through a common adapter interface. We incorporate upstream fixes from [PR #23](https://github.com/philippb/claude-homeassistant/pull/23) (template-aware device validation) and [PR #27](https://github.com/philippb/claude-homeassistant/pull/27) (sudo-friendly rsync), and keep fork-specific changes scoped to `.llm-agent/` to ease future upstream syncs.

## 🌟 Features

- **🤖 LLM-Agnostic**: Works with Claude Code, VS Code + GitHub Copilot, Ollama, llama.cpp, and more
- **🛡️ Multi-Layer Validation**: YAML syntax, entity references, and official HA validation
- **🔄 Safe Deployments**: Pre-push validation blocks invalid configs from reaching HA
- **🔍 Entity Discovery**: Tools to explore and search available entities
- **⚡ Automated Hooks**: Validation runs automatically via `.llm-agent/hooks/`
- **📊 Entity Registry Integration**: Real-time validation against your actual HA setup

## 🚀 Quick Start

### 1. Clone & Setup

```bash
git clone git@github.com:gooselabz/haa.git
cd haa
make setup  # Creates Python venv and installs dependencies
```

### 2. Configure Connection

```bash
cp .env.example .env
# Edit .env with your Home Assistant details
```

Required `.env` variables:

```bash
HA_TOKEN=your_home_assistant_long_lived_access_token
HA_URL=http://your_homeassistant_host:8123
HA_HOST=your_homeassistant_host   # SSH hostname
HA_REMOTE_PATH=/config/

# Set to 1 if your SSH user requires sudo for rsync on the HA host
USE_RSYNC_SUDO=0
```

**Recommended**: Install the [Advanced SSH &amp; Web Terminal](https://github.com/hassio-addons/addon-ssh) add-on for Home Assistant.

### 3. Pull Your Configuration

```bash
make pull  # Downloads your actual HA config into config/
```

### 4. Work with Your Configuration

- Edit configs locally with full validation
- Use your preferred LLM to create automations (see [LLM Integration](#-llm-integration))
- Validation hooks automatically check syntax and entity references

### 5. Push Changes

```bash
make push  # Validates then uploads to HA
```

## ⚙️ Prerequisites

- **Python 3.11+**
- **SSH access** to your Home Assistant host
- **make** (see below for installation)

### Installing make

**macOS:**

```bash
xcode-select --install
```

**Linux (Debian/Ubuntu):**

```bash
sudo apt install build-essential
```

**Windows:**

- WSL (recommended), or
- Git Bash, or
- `choco install make`

## 📁 Project Structure

```
haa/
├── .llm-agent/                # LLM hooks & adapters
│   ├── hooks/                 # Validation hooks (YAML format, pre-push)
│   └── adapters/              # LLM-specific adapters
│       ├── claude-code/
│       ├── llama-cpp/
│       ├── ollama/
│       └── vscode-copilot/
├── config/                    # Home Assistant configuration (synced from HA)
│   ├── configuration.yaml
│   ├── automations.yaml
│   ├── scripts.yaml
│   └── .storage/              # Entity registry
├── tools/                     # Validation & utility scripts
│   ├── run_tests.py           # Main test runner
│   ├── yaml_validator.py      # YAML syntax validation
│   ├── reference_validator.py # Entity reference validation
│   ├── ha_official_validator.py
│   └── entity_explorer.py     # Entity discovery
├── .env.example               # Environment template
├── Makefile                   # Core commands
├── HAA.md                     # Architecture & conventions
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
└── README.md
```

## 🛠️ Available Commands

### Configuration Management

```bash
make pull      # Pull latest config from Home Assistant
make push      # Push local config to HA (with validation)
make backup    # Create timestamped backup
make validate  # Run all validation tests
```

### Entity Discovery

```bash
make entities                           # Show entity summary
make entities ARGS='--domain climate'   # Climate entities only
make entities ARGS='--search motion'    # Search for motion sensors
make entities ARGS='--area kitchen'     # Kitchen entities only
make entities ARGS='--full'            # Complete detailed output
```

### Individual Validators

```bash
. venv/bin/activate
python tools/yaml_validator.py         # YAML syntax only
python tools/reference_validator.py    # Entity references only
python tools/ha_official_validator.py  # Official HA validation
```

### Natural Language Example

**Prompt**: "Turn off all lights at midnight on weekdays"

**Generated YAML**:

```yaml
- id: weekday_midnight_lights_off
  alias: "Weekday Midnight Lights Off"
  trigger:
    - platform: time
      at: "00:00:00"
  condition:
    - condition: time
      weekday: [mon, tue, wed, thu, fri]
  action:
    - service: light.turn_off
      target:
        entity_id: all
```

## 🔧 Validation System

The system provides three layers of validation:

### 1. YAML Syntax Validation

- Validates YAML syntax with HA-specific tags (`!include`, `!secret`, `!input`)
- Checks file encoding (UTF-8 required)
- Validates basic HA file structures

### 2. Entity Reference Validation

- Verifies all entity references exist in your HA instance
- Checks device and area references
- Warns about disabled entities
- Extracts entities from Jinja2 templates

### 3. Official HA Validation

- Uses Home Assistant's own validation tools
- Most comprehensive check available
- Catches integration-specific issues

## 🤖 LLM Integration

haa provides a unified adapter interface so any LLM can safely collaborate on Home Assistant configurations. Core workflow:

1. **Describe** the automation you want in natural language
2. **LLM drafts** YAML (or edits existing files)
3. **Validate** with `make validate`
4. **Deploy** with `make push` once validation passes

### Automated Hooks

The `.llm-agent/hooks/` directory contains shared hooks that:

- Format YAML files after edits
- Run validation before push operations
- Block invalid configs from reaching HA

### Adapter: Claude Code

Full integration with Anthropic's Claude Code desktop app.

**Setup**: Open the repo in Claude Code; reference `.llm-agent/adapters/claude-code/adapter.json` for hooks and settings.

**Workflow**:

1. Describe the automation ("add a dawn lights-off routine")
2. Claude proposes edits
3. Run `make validate` in terminal
4. Deploy with `make push`

See [`.llm-agent/adapters/claude-code/README.md`](.llm-agent/adapters/claude-code/README.md) for details.

### Adapter: VS Code + GitHub Copilot

Use Copilot Chat or inline completions within VS Code.

**Prerequisites**: VS Code 1.95+ with GitHub Copilot and Copilot Chat extensions.

**Workflow**:

1. Open the folder in VS Code
2. Reference `.llm-agent/adapters/vscode-copilot/adapter.json` for hooks and validation rules
3. Generate or modify automations; run validation in the integrated terminal
4. Deploy via `make push`

See [`.llm-agent/adapters/vscode-copilot/copilot-instructions.md`](.llm-agent/adapters/vscode-copilot/copilot-instructions.md) for details.

### Adapter: Ollama

Run haa workflows with local models served by [Ollama](https://ollama.ai).

**Prerequisites**: Ollama installed with a code-capable model (e.g., `llama3`, `phi3`, `mistral`).

**Workflow**:

1. Prime your model with the system prompt from `.llm-agent/adapters/ollama/prompts/`
2. Have the model draft YAML changes
3. Apply patches manually, then run `make validate`
4. Deploy with `make push`

See [`.llm-agent/adapters/ollama/README.md`](.llm-agent/adapters/ollama/README.md) for details.

### Adapter: llama.cpp

Self-hosted inference via [llama.cpp](https://github.com/ggerganov/llama.cpp) CLI or server.

**Prerequisites**: llama.cpp build with a downloaded model (e.g., `llama-3.1-70b-instruct.Q4_K_M.gguf`).

**Workflow**:

1. Prime the model with a system prompt describing haa conventions
2. Request a plan before edits
3. Apply patches, validate with `make validate`
4. Deploy with `make push`

See [`.llm-agent/adapters/llama-cpp/README.md`](.llm-agent/adapters/llama-cpp/README.md) for details.

## 📊 Entity Discovery

The entity explorer helps you understand what's available:

```bash
# Find all motion sensors
. venv/bin/activate && python tools/entity_explorer.py --search motion

# Show all climate controls
. venv/bin/activate && python tools/entity_explorer.py --domain climate

# Kitchen devices only
. venv/bin/activate && python tools/entity_explorer.py --area kitchen
```

## 🔒 Security & Best Practices

- **Secrets Management**: `secrets.yaml` is excluded from validation
- **SSH Authentication**: Uses SSH keys for secure HA access
- **No Credentials Stored**: Repository contains no sensitive data
- **Pre-Push Validation**: Prevents broken configs from reaching HA
- **Backup System**: Automatic timestamped backups before changes

## 🐛 Troubleshooting

### Validation Errors

1. Check YAML syntax first: `. venv/bin/activate && python tools/yaml_validator.py`
2. Verify entity references: `. venv/bin/activate && python tools/reference_validator.py`
3. Check HA logs if official validation fails

### SSH Connection Issues

1. Test connection: `ssh your_homeassistant_host`
2. Check SSH key permissions: `chmod 600 ~/.ssh/your_key`
3. Verify SSH config in `~/.ssh/config`

### Missing Dependencies

```bash
. venv/bin/activate
pip install homeassistant voluptuous pyyaml jsonschema requests
```

## 🔧 Configuration

### Environment Variables

Create `.env` from the template:

```bash
cp .env.example .env
```

| Variable              | Required | Description                                                |
| --------------------- | -------- | ---------------------------------------------------------- |
| `HA_TOKEN`          | Yes      | Home Assistant long-lived access token                     |
| `HA_URL`            | Yes      | HA instance URL (e.g.,`http://homeassistant.local:8123`) |
| `HA_HOST`           | Yes      | SSH hostname for rsync operations                          |
| `HA_REMOTE_PATH`    | No       | Remote config path (default:`/config/`)                  |
| `USE_RSYNC_SUDO`    | No       | Set to `1` if SSH user needs sudo for rsync              |
| `LOCAL_CONFIG_PATH` | No       | Local config directory (default:`config/`)               |
| `BACKUP_DIR`        | No       | Backup directory (default:`backups`)                     |
| `VENV_PATH`         | No       | Python venv path (default:`venv`)                        |
| `TOOLS_PATH`        | No       | Tools directory (default:`tools`)                        |

### Adapter Settings

Each adapter has its own `adapter.json` with hooks, validation, and development settings. Example from `.llm-agent/adapters/vscode-copilot/adapter.json`:

```json
{
  "hooks": {
    "enabled": true,
    "posttooluse": [".llm-agent/hooks/yaml-formatter.sh", ".llm-agent/hooks/posttooluse-ha-validation.sh"],
    "pretooluse": [".llm-agent/hooks/pretooluse-ha-push-validation.sh"]
  },
  "validation": {
    "enabled": true,
    "auto_run": true,
    "block_invalid_push": true
  }
}
```

Hook scripts live in `.llm-agent/hooks/` and are shared across adapters.

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:

- Keeping divergence low with upstream
- Running tests before PRs
- Adapter-specific documentation

## 📄 License

Apache 2.0 — same as upstream.

## 🙏 Acknowledgments

- [philippb/claude-homeassistant](https://github.com/philippb/claude-homeassistant) — upstream project
- [Home Assistant](https://home-assistant.io) — the platform
- The HA community for validation best practices
