# Home Assistant Blueprints

Automation blueprints for Home Assistant.

| Blueprint | What it does | Import |
|---|---|---|
| **Motion Light Plus** | Lights on with motion or occupancy from one or more sensors, optionally only between set times or between sunset and sunrise (with offsets). Once every sensor has cleared and a delay has passed, turn the lights off or change them to a different brightness / colour / temperature. Brightness and colour (temperature or RGB) are set separately for "on" and "after motion". | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fmotion_light_plus.yaml) |
| **Button actions** | Single / double / long press actions for a Matter or Shelly button exposed as an `event` entity. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fbutton_actions.yaml) |
| **Dual button actions** | Single / double / long press actions for **both** buttons of a two-button remote (e.g. IKEA BILRESA) where each button is its own `event` entity. One automation instead of two. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fdual_button_actions.yaml) |
| **Wall switch toggles light** | A detached wall switch (e.g. Shelly input) toggles lights on every flip. Ignores `unavailable`/`unknown`, so reboots and Wi-Fi drops don't flip lights. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fswitch_toggle.yaml) |

## Install

Click an **Import** badge above, or in Home Assistant go to
**Settings → Automations & scenes → Blueprints → Import blueprint** and paste the file's GitHub URL.

Blueprints imported this way include a `source_url`, so you can later use
**⋮ → Re-import blueprint** to pick up updates.

## Examples

```yaml
# Motion Light Plus: only between sunset and sunrise, 100% warm white, off 5 min after motion stops
use_blueprint:
  path: drr0586/motion_light_plus.yaml
  input:
    trigger_section:
      motion_sensors:
        - binary_sensor.kitchen_motion_occupancy
    condition_section:
      condition_mode: sun
    on_section:
      lights: { entity_id: [light.kitchen_spots] }
      on_brightness: 100
      on_color_mode: color_temp
      on_color_temp: 2700
    off_section:
      off_delay: { hours: 0, minutes: 5, seconds: 0 }
      off_action: turn_off
```

```yaml
# Outside light: full brightness on motion, then dim to amber after 5 min instead of turning off
use_blueprint:
  path: drr0586/motion_light_plus.yaml
  input:
    trigger_section:
      motion_sensors:
        - binary_sensor.front_door_person
    on_section:
      lights: { entity_id: light.outside }
      on_brightness: 100
      on_color_mode: color_temp
      on_color_temp: 5000
    off_section:
      off_delay: { hours: 0, minutes: 5, seconds: 0 }
      off_action: change
      off_transition: 20
      change_brightness: 30
      change_color_mode: rgb
      change_rgb: [217, 80, 0]
```

```yaml
# Button: single = scene, double = full brightness, long = dimmed scene
use_blueprint:
  path: drr0586/button_actions.yaml
  input:
    button: event.bedroom_button_button_1
    single_press:
      - action: scene.turn_on
        target: { entity_id: scene.bedroom_relax }
    double_press:
      - action: light.turn_on
        target: { entity_id: light.bedroom_ceiling }
        data: { brightness_pct: 100 }
    long_press:
      - action: scene.turn_on
        target: { entity_id: scene.bedroom_dimmed }
```

```yaml
# Two-button remote (e.g. IKEA BILRESA): button 1 has all three presses, button 2 just turns a light off
use_blueprint:
  path: drr0586/dual_button_actions.yaml
  input:
    button_1_section:
      button_1: event.bedroom_button_button_1
      button_1_single:
        - action: scene.turn_on
          target: { entity_id: scene.bedroom_relax }
      button_1_double:
        - action: light.turn_on
          target: { entity_id: light.bedroom_ceiling }
          data: { brightness_pct: 100 }
      button_1_long:
        - action: scene.turn_on
          target: { entity_id: scene.bedroom_dimmed }
    button_2_section:
      button_2: event.bedroom_button_button_2
      button_2_single:
        - action: light.turn_off
          target: { entity_id: light.bedroom }
```

> The `path:` depends on where the blueprint is stored. Imported from GitHub it's usually
> `drr0586/<file>.yaml`; copied by hand it's whatever folder you put it in under
> `/config/blueprints/automation/`.
