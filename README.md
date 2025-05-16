# esphome

ESPHome yaml configuration for some automations around the house.

### Makup Air System (muas)
- Prevents home depressurization while using exhaust hood by controling a fan that replaces exhausted air with outside air.
- Opens a 24V damper using a relay when exhaust hood is in use
- Senses 0-10V signal voltage of an ECM exhaust fan to create a proportional output voltage using PWM for a 0-10V ECM makeup air fan.
- Can be used in any exhaust system by determining maximum intake fan speed during maximum exhaust fan speed that results in a 0kPa differential air pressure using a manometer.
- Future Idea: Because intake and exhaust fans have different fan curves, create a mapping at voltage ranges to replace the linear relationship and account for static pressure differences.

