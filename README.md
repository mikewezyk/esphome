# ESPHome home automation configs

[![Validate ESPHome configs](../../actions/workflows/validate.yml/badge.svg)](../../actions/workflows/validate.yml)
[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-lightgrey.svg)](LICENSE.md)

Three [ESPHome](https://esphome.io) projects for ESP32 boards that integrate with
[Home Assistant](https://www.home-assistant.io):

| Project | File | What it does |
|---|---|---|
| [Makeup air system](#makeup-air-system-muas) | `muas.yaml` | Runs a makeup-air fan and damper in proportion to a range hood's speed, to avoid depressurizing the house |
| [ERV controller](#erv-controller-erv) | `erv.yaml`, `erv_sen55.yaml` | Controls an energy recovery ventilator and shuts it off when outdoor air quality is bad |
| [Sprinkler controller](#sprinkler-controller-sprinkler) | `sprinkler.yaml` | 3-zone lawn irrigation with a shared pump |

Each config is self-contained, validated in CI, and uses substitutions at the top of the
file so you can change names, pins, and tuning values without digging through the YAML.

> **Safety:** these projects switch mains/24V loads and water. Follow local electrical
> code and treat the firmware as a convenience, not a safety device. See
> [SECURITY.md](SECURITY.md).

## Quick start

1. Install ESPHome (`pip install esphome`) or use the Home Assistant ESPHome add-on.
2. Clone this repo and create your secrets file:
   ```bash
   cp secrets.example.yaml secrets.yaml
   # then edit secrets.yaml; generate keys with the commands inside it
   ```
3. Edit the `substitutions:` block at the top of the config you want (name, pins, tuning).
4. Check, flash, and watch:
   ```bash
   esphome config muas.yaml      # validate
   esphome run muas.yaml         # compile + upload (USB the first time, OTA after)
   ```
5. Home Assistant will discover the device; enter its API encryption key when asked.

`secrets.yaml` is gitignored. Every device uses its own API key, OTA password, and
fallback-hotspot password.

> **Upgrading an older install:** earlier versions had empty OTA passwords. The first
> flash of this version must be done with the *old* firmware's settings (or by USB);
> after that, OTA requires the password from `secrets.yaml`.

## Makeup air system (`muas`)

Prevents depressurization when an exhaust hood runs. The ESP32 reads the hood's
0-10 V ECM speed signal, opens a 24 V damper, then drives a 0-10 V ECM intake fan at a
proportional speed.

**Behavior**
- Hood signal below `exhaust_fan_threshold_low` (default 2 V), or an unreadable ADC: fan off, damper closed.
- Otherwise: open damper, wait `damper_open_delay` (10 s), then run the fan at
  `exhaust_voltage / max_voltage * makeup_air_fan_max_speed` percent.
- If the hood is switched off during the damper delay, the pending fan start is cancelled.
- After a reboot the damper and fan start **off**.

**Wiring (NodeMCU-32S)**

| Function | Pin | Notes |
|---|---|---|
| Exhaust fan 0-10 V signal | `GPIO36` | Through a voltage divider so the pin never exceeds 3.3 V |
| 0-10 V PWM output to intake fan | `GPIO25` | LEDC 3 kHz into a PWM-to-0-10 V converter |
| Damper relay | `GPIO26` | Active low, switches the 24 V damper |

**Calibrating for your own setup**
1. Set `adc_multiplier` for your divider: `(R_top + R_bottom) / R_bottom * (10 / 3.3)`.
   The default `3.1757` is `(49254 / 47000) * (10 / 3.3)`. Compare *Exhaust Hood Voltage*
   to a multimeter.
2. With the hood at full speed, raise `makeup_air_fan_max_speed` until a manometer reads
   0 Pa between indoors and outdoors.
3. Idea for the future: replace the linear map with a piecewise curve, since the two
   fans have different curves.

## ERV controller (`erv`)

Controls an ERV (power and boost relays) and watches indoor air with a Sensirion SEN55.
If particulates, VOC, or NOx stay above a threshold for 60 s, the ERV is switched off;
it resumes automatically once readings drop below the (lower) resume thresholds.

**Entities exposed to Home Assistant**
- Switches: `erv_power`, `erv_boost`, *ERV AQ Protection* (master enable)
- Numbers: shutoff/resume thresholds for PM2.5, VOC, and NOx; SEN55 temperature and humidity offsets
- Sensors: PM1/2.5/4/10, per-size-range PM (disabled by default), temperature, humidity, VOC, NOx, NowCast AQI, VOC quality text
- Button and service: *Clean SEN55* / `sen55_clean` (fan autoclean)

**Fail-safe design:** the ERV is never left off because of a sensor problem. If the SEN55
is missing or returns invalid data, protection is skipped and the ERV keeps running. After
a reboot the ERV turns on unless a healthy sensor reports an over-threshold reading.

**Wiring (NodeMCU-32S)**

| Function | Pin |
|---|---|
| ERV power relay (active low) | `GPIO25` |
| ERV boost relay | `GPIO26` |
| SEN55 I2C SDA / SCL (address `0x69`) | `GPIO21` / `GPIO22` |

Set `SEN55 Temperature Offset` to compensate for self-heating once the sensor is in its enclosure.

## Sprinkler controller (`sprinkler`)

Three zones (Backyard Corner, Center, Drip) share one pump using ESPHome's
[`sprinkler`](https://esphome.io/components/sprinkler.html) controller, with auto-advance
and reverse switches. Each zone defaults to 15 minutes and is adjustable from Home Assistant.

| Function | Pin (all active low) |
|---|---|
| Pump | `GPIO32` |
| Zone 1 Corner / 2 Center / 3 Drip | `GPIO33` / `GPIO25` / `GPIO26` |

All relays start **off** after a reboot or power loss, so a crash can't leave water running.
Rename zones or change pins in the file; to add a zone, copy a block under `valves:` and add a switch.

## Repository layout

```
muas.yaml            Makeup air system
erv.yaml             ERV controller (includes erv_sen55.yaml as a package)
erv_sen55.yaml       SEN55 sensor config, adapted from Apollo AIR-1
sprinkler.yaml       Sprinkler controller
secrets.example.yaml Template for your (gitignored) secrets.yaml
SECURITY.md          Security model and how to report issues
.github/workflows/   CI: validates and compiles every config
```

## Contributing

Issues and pull requests are welcome, particularly for other boards, other fan/ERV
hardware, and wiring diagrams. Please run `esphome config <file>` before submitting;
CI also compiles every config.

## License

[CC BY-NC-SA 4.0](LICENSE.md). `erv_sen55.yaml` is adapted from
[Apollo Automation AIR-1](https://github.com/ApolloAutomation/AIR-1) (same license).
