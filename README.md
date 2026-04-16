# esphome

ESPHome yaml configuration for some automations around the house.

### Makeup Air System (muas)
- Prevents home depressurization while using exhaust hood by controlling a fan that replaces exhausted air with outside air.
- Opens a 24V damper using a relay when exhaust hood is in use.
- Senses 0-10V signal voltage of an ECM exhaust fan to create a proportional output voltage using PWM for a 0-10V ECM makeup air fan.
- Can be used in any exhaust system by determining maximum intake fan speed during maximum exhaust fan speed that results in a 0kPa differential air pressure using a manometer.
- Future Idea: Because intake and exhaust fans have different fan curves, create a mapping at voltage ranges to replace the linear relationship and account for static pressure differences.

### ERV (erv)
- Controls an Energy Recovery Ventilator to manage fresh air intake and exhaust.
- Two relay outputs for ERV power and boost mode.
- Monitors indoor air quality using a Sensirion SEN55 sensor over I2C: PM1.0, PM2.5, PM4.0, PM10.0, temperature, humidity, VOC index, and NOx index.
- Exposes per-particle-size PM breakdowns and a VOC quality text label to Home Assistant.
- Configurable temperature and humidity offsets to compensate for sensor self-heating.
- Supports manual SEN55 fan autoclean via a Home Assistant service call.

### Sprinkler (sprinkler)
- Controls a 3-zone lawn irrigation system (Backyard Corner, Center, and Drip zones).
- Uses ESPHome's sprinkler controller platform with auto-advance and reverse support.
- Each zone runs for 15 minutes with a shared pump relay and per-zone enable switches.
- Zones and pump use inverted GPIO outputs for relay board compatibility.

