# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Home Assistant configuration repository running version 2025.10.4. The setup uses a modular architecture with split configuration files, custom templates, packages, and YAML-based Lovelace dashboards.

## Configuration Architecture

### Core Structure

- `configuration.yaml` - Main entry point using includes for modularity
- `automations.yaml` - All automations (garage monitoring, TV automation, Roku controls)
- `scripts.yaml` - Reusable scripts (lighting toggles, TV toast notifications)
- `secrets.yaml` - Credential storage (never commit real secrets)

### Modular Components

**Packages** (`packages/`): Self-contained configuration bundles using `!include_dir_named`
- Currently contains `garage.yaml` (empty placeholder for future garage-related config)
- Packages allow grouping related entities, automations, and scripts together

**Custom Templates** (`custom_templates/`): Template entities using `!include_dir_merge_list`
- `all_lights.yaml` - Template switch that aggregates `light.lighting_lights` and `switch.lighting_switches`
- `garage_cover.yaml` - Template cover entity that wraps garage door relay and inverts reed switch logic
- `steps.yaml` - Step tracking templates
- Note: Files must be YAML list items (start with `-`) due to `merge_list` directive

**Dashboards** (`dashboards/`): YAML mode Lovelace UI
- `main.yaml` - Main dashboard entry point with view includes
- `home.yaml` - Home view with person cards, camera, climate, garage controls
- `lights.yaml` - Lighting controls
- `cameras.yaml` - Camera views
- `roku.yaml` - Media player controls

### Custom Frontend Resources

The dashboard uses several HACS custom cards (loaded via `main.yaml` resources):
- `button-card` - Advanced button customization with JavaScript templates
- `stack-in-card` - Card stacking layouts
- `mini-graph-card` - Compact graphs
- `swipe-card` - Swipeable card carousels
- `card-mod` - CSS styling modifications
- `mushroom` - Minimalist card designs

## Key System Behaviors

### Garage Door System

The garage door uses an inverted logic system:
- Physical sensor: `binary_sensor.garage_door_closed_raw` (ON = closed, OFF = open)
- Template sensor: `binary_sensor.garage_door_open` inverts the raw sensor
- Template cover: `cover.garage_door` provides garage door interface
- Momentary relay: `switch.garage_door_relay` - all actions (open/close/stop) trigger the same relay pulse

**Garage Automations**:
- Daytime (before 10 PM): Single notification after 10 minutes open
- Nighttime (10 PM - 6 AM): Critical alerts every 10 minutes with TV toast if TV is on
- Push notification actions: "Close now" and "Mute 30 min"
- Mute state: `input_boolean.garage_night_nag_mute` (auto-resets at 6 AM)

### TV/Theater Automation

**Movie Mode** (triggers on Netflix/Prime Video/Plex):
- Snapshots current theater lights to `scene.theater_prev_lights`
- Dims `light.theater_main_back` and `light.theater_theater_area` to 10%
- Switches TV audio to external ARC
- On pause: Brightens lights to 30%
- On play: Dims lights back to 10%
- On source change or TV off: Restores snapshot

**Roku Controls**:
- Bedtime auto-off at 11:30 PM if not playing
- Script shortcuts for YouTube TV, Netflix, Prime Video

### Lighting System

- Group entities: `light.lighting_lights` and `switch.lighting_switches`
- Template switch `switch.all_lighting` shows ON if any light is on
- Script `lighting_toggle_all` intelligently turns all off if any on, or all on if all off

## Configuration Validation

Check configuration before restarting Home Assistant:
```bash
# From Home Assistant host (not available in this container environment)
ha core check
```

## Development Commands

This is a running Home Assistant container, not a development environment. Configuration changes take effect after:

1. Reload YAML configuration (for most changes):
   - Use Home Assistant UI: Developer Tools → YAML → Reload specific sections
   - Or restart Home Assistant via UI or `ha core restart` (from host)

2. For template changes in `custom_templates/`:
   - Reload "Template Entities" in Developer Tools

3. For dashboard changes in `dashboards/`:
   - Changes auto-reload or refresh browser

## Entity Naming Patterns

When referencing entities in automations/scripts:
- Garage: `cover.garage_door`, `binary_sensor.garage_door_open`, `switch.garage_door_relay`
- Lighting: `light.lighting_lights`, `switch.lighting_switches`, `switch.all_lighting`
- Climate: `climate.upstairs`, `climate.downstairs`
- Media: `media_player.lg_webos_tv`, `media_player.roku_family_room`, `media_player.spotify_bob`
- Person tracking: `person.bobby_smith`, `device_tracker.s_word`, `sensor.s_word_battery_level`
- Theater lights: `light.theater_main_back`, `light.theater_theater_area`
- Notifications: `notify.mobile_app_sword`, `notify.lg_webos_tv_oled83c1pua`

## Important Integration Details

**LG WebOS TV** (`media_player.lg_webos_tv`):
- Supports toast notifications via `notify.lg_webos_tv_oled83c1pua`
- Sound output switching via `webostv.select_sound_output`
- Monitors `state` and `source` attribute for automation triggers

**Button Card Templates**: Use JavaScript template syntax `[[[ return ... ]]]` for dynamic content and styling

**Template Syntax**: Mix of Jinja2 (`{{ }}`, `{% %}`) and JavaScript (`[[[...]]]`) depending on context:
- YAML templates/conditions: Jinja2
- Button card custom fields: JavaScript
- Card-mod styles: JavaScript within Jinja2 wrappers

## File Organization Best Practices

- Add entity-specific config to `packages/` for self-contained features
- Use `custom_templates/` for template sensors, binary_sensors, switches, covers
- Keep automations in `automations.yaml` unless tightly coupled to a package
- Dashboard views should be separate files included from `main.yaml`
- Never commit sensitive data in `secrets.yaml` (use `!secret` references in config files)
