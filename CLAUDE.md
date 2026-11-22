# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A collection of Home Assistant blueprints (YAML), published at https://github.com/drr0586/ha_blueprints (`main`). Blueprints live in `blueprints/automation/`: `motion_light_plus.yaml` ("Motion Light Plus": motion-triggered lighting with time/sun conditions and an off-or-change-state step after a delay), `button_actions.yaml` (one button), `dual_button_actions.yaml` (two buttons, each its own event entity, one automation) and `switch_toggle.yaml`. All except `dual_button_actions.yaml` (new; syntax-checked only, not yet imported into HA) match what is deployed in Home Assistant; the HA copies of button/switch are re-serialised by HA on save, so they differ from the repo in formatting only. `README.md` has a table with an import badge per blueprint; add a row when adding a blueprint. There is no build, lint or test tooling. The git sandbox cannot write `.git/config`, so repo setup and config changes have to be run by the user.

## Workflow

- **Syntax check:** parse with PyYAML using a loader that registers the `!input` tag (a plain `yaml.safe_load` fails on it). PyYAML is not importable in the sandbox; use a throwaway venv in the scratchpad (`python3 -m venv`, `pip install pyyaml`) and delete it afterwards. This only proves valid YAML, not that Home Assistant accepts the schema.
- **Deploying:** files are copied by the user into `<HA config>/blueprints/automation/<folder>/` (Samba, SSH or File editor add-on), then reloaded. The HA MCP server cannot write files or manage blueprints, and it only sees entities exposed to Assist. Automations show only on/off state, not their configuration; ask the user to paste the YAML from the automation editor to inspect it.
- **Real validation** happens when the user imports the file into HA; schema errors are reported there (and in Settings → System → Logs).

## Blueprint gotchas

- `min_version` must be nested: `blueprint.homeassistant.min_version` (not `blueprint.min_version`). Collapsible input `sections` require 2024.6.0+.
- The automation's name is chosen by the user, not the blueprint; it cannot be forced from YAML.
- Sun condition offsets (`after_offset`/`before_offset`) take `HH:MM:SS` strings with optional leading minus, so they are `text` selectors rather than duration selectors (which can't be negative).
- `!input` is plain substitution into the config, so inputs used inside templates are first copied into top-level `variables:` (e.g. `condition_mode`, `on_color_mode`).

## `blueprints/automation/motion_light_plus.yaml` design

- `mode: restart`: new motion during the post-motion delay cancels and restarts the run. The wait-for-clear step uses `wait_template` that checks all selected sensors are off, then `delay`.
- Conditions (always / time / sun) are only evaluated when motion triggers; they gate turning the light on, not turning it off.
- Colour handling is a `choose` on colour mode (none / `color_temp` kelvin / `rgb`), duplicated for the "on" step and the "change state" step.
