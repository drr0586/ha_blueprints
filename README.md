# Home Assistant Blueprints

Automation blueprints for Home Assistant.

| Blueprint | What it does | Import |
|---|---|---|
| **Motion light (plus)** | Lights on with motion, off (or dimmed when dark) after motion stops. Optional "only when dark" (minutes around sunset/sunrise) or "between set times". Brightness / colour temperature optional. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fmotion_light_plus.yaml) |
| **Button actions** | Single / double / long press actions for a Matter or Shelly button exposed as an `event` entity. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fbutton_actions.yaml) |
| **Wall switch toggles light** | A detached wall switch (e.g. Shelly input) toggles lights on every flip. Ignores `unavailable`/`unknown`, so reboots and Wi-Fi drops don't flip lights. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdrr0586%2Fha_blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fswitch_toggle.yaml) |

## Install

Click an **Import** badge above, or in Home Assistant go to
**Settings → Automations & scenes → Blueprints → Import blueprint** and paste the file's GitHub URL.

Blueprints imported this way include a `source_url`, so you can later use
**⋮ → Re-import blueprint** to pick up updates.

## Examples

```yaml
# Motion light: only when dark, 100% warm white, off 5 min after motion stops
use_blueprint:
  path: drr0586/motion_light_plus.yaml
  input:
    motion_entity: binary_sensor.kitchen_motion_occupancy
    lights: { entity_id: [light.kitchen_spots] }
    when: dark
    brightness: 100
    color_temp: 2700
    no_motion_wait: { minutes: 5 }
```

```yaml
# Outside light: full brightness on motion, then dim to amber when dark
use_blueprint:
  path: drr0586/motion_light_plus.yaml
  input:
    motion_entity: binary_sensor.front_door_person
    lights: { entity_id: light.outside }
    brightness: 100
    color_temp: 5000
    after_motion: dim_when_dark
    dim_brightness: 30
    dim_color: [217, 80, 0]
    transition: 20
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

> The `path:` depends on where the blueprint is stored. Imported from GitHub it's usually
> `drr0586/<file>.yaml`; copied by hand it's whatever folder you put it in under
> `/config/blueprints/automation/`.
