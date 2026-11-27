# Security Policy

## Reporting a vulnerability

Please report security issues privately through GitHub's
[private vulnerability reporting](../../security/advisories/new) rather than a public
issue. Include the affected file, what an attacker could do, and steps to reproduce.

## Security model

These configs are intended for devices on a trusted home LAN, managed by Home Assistant.

| Surface | Protection |
|---|---|
| Home Assistant API | Noise encryption with a per-device key (`*_api_key`) |
| OTA firmware updates | Per-device password (`*_ota_password`) |
| Fallback hotspot | WPA2 with a per-device password (`*_ap_password`) |
| Wi-Fi client | `min_auth_mode: WPA2` (no WPA1/open networks) |
| Secrets | Kept in `secrets.yaml`, which is gitignored; only `secrets.example.yaml` is committed |

Recommended practices if you reuse these configs:

- Generate a **unique** key/password for every device. Never reuse the examples.
- Put IoT devices on their own VLAN/SSID with no inbound access from the internet.
- Do not expose the ESP devices (port 6053 API, OTA) to the internet.
- If a secret is ever committed, rotate it immediately; removing it from git history
  is not enough.
- Relay outputs fail safe: they default to **off** after a reboot or power loss.

## Safety

These projects switch mains/24V loads and water. Wire according to local electrical
code, fuse appropriately, and treat all firmware as safety-adjacent but **not**
safety-rated. Do not rely on it as the only protection against flooding, overpressure,
or fire.
