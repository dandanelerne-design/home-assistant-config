# Copilot / AI Agent Instructions for this repository ✅

Purpose: short, actionable notes to help AI coding agents be productive editing this Home Assistant configuration repo.

Big picture
- This repo is a Home Assistant configuration (not a Python application). Primary runtime is Home Assistant OS / Core. See `configuration.yaml` (root) for global structure.
- Key responsibilities:
  - `configuration.yaml` is the entry point and uses `packages: !include_dir_named packages` and `lovelace: mode: yaml` to assemble the system.
  - `packages/` contains modular configuration packages (grouped sensors, automations, integrations). Edit a package file to add a new integration or sensor.
  - `include/` contains Lovelace fragments used by `tablet.yaml`, `mobile.yaml`, and `smart-clock.yaml`. Reusable UI pieces (headers, cards) live here.
  - `custom_components/` contains local Home Assistant integrations (Python). These follow HA integration patterns (manifest.json, `__init__.py`, `config_flow.py`, platforms like `sensor.py`). Example: `custom_components/garbage_collection`.
  - Automations are primarily in Node-RED: `automations/node-RED/*.json` (flows are exported/imported JSON).
  - Frontend resources are managed with HACS and referenced under `lovelace.resources` (e.g., `/hacsfiles/...`).

Project-specific conventions and patterns
- YAML include pattern: the repo relies heavily on `!include` & `!include_dir_named`. Keep fragments small and idempotent so they work when included by `configuration.yaml`.
- Secrets: credentials and private tokens use `!secret`. Locally required `secrets.yaml` is intentionally absent — do not commit secrets.
- Lovelace setup: dashboards are stored in separate YAML files (`tablet.yaml`, `tablet_ui.yaml` is a consolidated UI-mode fallback). Use `include/views/` for view fragments.
- Node-RED: flow files live under `automations/node-RED/` and are the source of truth for automation logic. Use Node-RED import/export to sync changes.
- Custom integration pattern examples:
  - Register services with `hass.services.async_register` (see `custom_components/garbage_collection/__init__.py`).
  - Config entry lifecycle methods are implemented (`async_setup_entry`, `async_remove_entry`, migrations in `async_migrate_entry`). Follow this pattern when adding integrations.
  - Use `manifest.json` to declare dependencies and requirements.

Debugging & developer workflows (practical steps)
- Config verification: use Home Assistant UI -> Configuration -> Server Controls -> "Check Configuration" before restarting.
- Reload vs restart:
  - UI-only changes (Lovelace YAML fragments): clear browser cache and reload to see JS resource changes. Some UI resources require a Home Assistant restart.
  - Custom components require a Home Assistant restart to pick up Python code changes. Use the UI restart, or restart the Supervisor/Host depending on your environment.
- Enabling debug logs: there are commented `logger:` sections in `configuration.yaml`. To debug a custom component, add e.g.:
  logger:
    default: info
    logs:
      custom_components.garbage_collection: debug
  Then check the Home Assistant log (Supervisor → System logs or `home-assistant.log`).
- Testing templates and sensors: use Developer Tools → Template editor and Developer Tools → States to evaluate templates and inspect entity state changes.
- Node-RED flow testing: import the JSON from `automations/node-RED/*.json` into Node-RED to run or simulate flows.

Files & locations to reference when making changes
- `configuration.yaml` — global settings, includes, logger examples, lovelace resources
- `packages/` — modular HA package files (grouping of sensors, switches, automations)
- `include/` — Lovelace card and view fragments used across dashboards
- `tablet.yaml`, `mobile.yaml`, `smart-clock.yaml`, `tablet_ui.yaml` — dashboard definitions
- `custom_components/` — local custom integrations to edit or extend
- `automations/node-RED/` — Node-RED flows (JSON) for automations
- `www/assets/` — static assets used by dashboards (images, icons, svgs)

What NOT to change without verification
- Don’t commit secrets or API keys (repo uses `!secret`).
- Avoid large, invasive Lovelace refactors without testing on a local HA instance or a dev instance—small iterated changes are easier to validate.

Examples to follow
- To add a small sensor: add a YAML file in `packages/` and reference any secrets needed via `!secret`.
- To add a service to an integration: mirror `custom_components/garbage_collection` approach: register service in `async_setup`, add schema validation using `voluptuous`, and write clear log messages.

Notes / Limitations
- There are no automated tests in the repo; rely on Home Assistant runtime checks and log output.
- The environment expected for running this code is a Home Assistant instance (Supervised / Core / OS). A local dev container or VM with Home Assistant installed is recommended for iterative development.

If anything above is ambiguous or you want more detailed examples (for instance: a checklist for creating a new custom component, or step-by-step Node-RED import), say which area to expand and I will iterate. 🙋‍♀️