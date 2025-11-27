# Home Topology Generation Prompt

Generate a comprehensive topology document for this Home Assistant instance.

## Pre-check

Before proceeding, confirm `/config` reflects the user's actual HA instance:
1. Check that `/config/.storage/` contains `core.device_registry` and `core.entity_registry`
2. Verify `configuration.yaml` exists and references real integrations
3. If config appears stale or missing, run `make pull` first

## Output

Generate `/.llm-agent/refs/hometopology.md` with these sections:

### 1. System Overview
- Platform version and installation type
- Zigbee coordinator (brand, model, channel)
- Z-Wave controller (if present)
- Thread/Matter border routers
- MQTT broker config
- Cloud integrations (Nabu Casa, vendor clouds)

### 2. Physical Structure
- Floors (if defined)
- Areas with room counts
- Device distribution per area

### 3. Labels and Categories
- Custom labels in use
- Device classes and categories
- Hidden/disabled entity patterns

### 4. Integrations & Add-ons
- Active integrations (count entities per integration)
- Installed add-ons
- Custom components (HACS or manual)

### 5. Entities
- Total entity count by domain
- Naming convention analysis
- Orphaned or unavailable entities

## Format

Use concise Markdown tables where appropriate. Keep the document under 500 lines for efficient LLM context consumption.
