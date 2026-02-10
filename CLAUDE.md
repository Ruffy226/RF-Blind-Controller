# CLAUDE.md - RF Blind Controller

This file provides guidance for AI assistants working on this repository.

## Project Overview

**RF-Blind-Controller** is an ESPHome-based project that uses a **Seeed Studio XIAO ESP32C6** microcontroller with 433MHz RF modules to control **Levolor motorized window blinds** via Home Assistant. The device captures RF codes from the existing Levolor remote and replays them to open, close, and stop the blinds.

## Hardware

### Microcontroller

- **Board:** Seeed Studio XIAO ESP32C6
- **Chip:** ESP32-C6 (RISC-V)
- **Framework:** ESP-IDF (Arduino framework is **not supported** for ESP32-C6 in ESPHome)
- **WiFi:** 2.4 GHz 802.11ax (WiFi 6) — sufficient for Home Assistant communication
- **Additional:** Bluetooth 5.3 LE, Zigbee/Thread capable
- **Battery:** Built-in LiPo charging circuit (key feature of the XIAO form factor)
- **Size:** 21mm x 17.5mm

### Pin Assignments

| Pin    | Function           | Notes                                    |
|--------|--------------------|------------------------------------------|
| GPIO2  | RF Receiver (RX)   | 433MHz receiver data pin, input with no pullup |
| GPIO3  | RF Transmitter (TX) | 433MHz transmitter data pin (planned)    |

### RF Modules

- **Frequency:** 433MHz (standard for Levolor remotes)
- **Receiver:** Connected to GPIO2 — used to capture/learn codes from the Levolor remote
- **Transmitter:** Planned for GPIO3 — used to replay captured codes to control blinds

### Target Device

- **Blinds:** Levolor motorized blinds with existing RF remote
- **Commands:** Up, Down, Stop (captured from remote)

## Software Stack

### Platform

- **ESPHome** running on **Home Assistant**
- **Home Assistant host:** Raspberry Pi (accessible at `192.168.4.222:8123`)
- **ESPHome version:** 2025.10.4+
- **Build framework:** ESP-IDF (not Arduino)

### Configuration

The project uses ESPHome YAML configuration. The main config file lives on the Raspberry Pi at:

```
/config/esphome/levolor-blind-controller.yaml
```

> **Note:** The file may also appear as `waveshare-esp32c6.yaml` if it was repurposed from a previous Waveshare board configuration. The ESPHome device name is `levolor-blind-controller`.

### ESPHome YAML Structure

```yaml
esphome:
  name: levolor-blind-controller
  friendly_name: Levolor Blind Controller

esp32:
  board: seeed_xiao_esp32c6
  variant: esp32c6
  framework:
    type: esp-idf          # MUST be esp-idf, NOT arduino

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Levolor-Blind Fallback"
    password: "fallback123"

api:                        # Home Assistant API (key auto-generated)

ota:
  - platform: esphome
    password: !secret ota_password

web_server:
  port: 80                  # Optional debug web interface

remote_receiver:            # RF Receiver for learning codes
  pin:
    number: GPIO2
    inverted: false
    mode:
      input: true
      pullup: false
  dump: all
  tolerance: 50%
  filter: 250us
  idle: 4ms

# remote_transmitter:       # Uncomment after capturing codes
#   pin: GPIO3
#   carrier_duty_percent: 100%
```

### Secrets

Secrets are stored in `/config/esphome/secrets.yaml` on the Raspberry Pi:

```yaml
wifi_ssid: "..."
wifi_password: "..."
ota_password: "..."
```

**Do not commit actual secret values to this repository.**

## Development Workflow

### Building and Flashing

1. **Edit config:** ESPHome dashboard → Edit YAML (or SSH into Pi and edit directly)
2. **Compile:** ESPHome dashboard → Install → (choose method)
3. **First flash (USB):** ESPHome dashboard → "Plug into this computer" → select `/dev/ttyACM0`
4. **Subsequent updates (OTA):** ESPHome dashboard → Install → Wirelessly

### Bootloader Mode (if USB flash fails)

1. Unplug the XIAO
2. Hold the tiny **BOOT** button (not RESET)
3. While holding BOOT, plug in USB
4. Keep holding for 5 seconds
5. Release and immediately flash

### Build Output Locations (on Raspberry Pi)

```
/config/.esphome/build/levolor-blind-controller/.pioenvs/levolor-blind-controller/
├── firmware.bin              # Standard firmware
├── firmware.factory.bin      # Full factory image (for first flash)
└── firmware.ota.bin          # OTA update image
```

### Manual Flash via esptool

```bash
esptool.py --chip esp32c6 --port /dev/ttyACM0 --baud 460800 \
  write_flash 0x0 firmware.factory.bin
```

### Resource Usage (last build)

- **RAM:** 11.4% (37,472 / 327,680 bytes)
- **Flash:** 51.6% (946,891 / 1,835,008 bytes)

## Project Status

### Completed

- [x] ESPHome YAML configuration written
- [x] Firmware compiles successfully
- [x] WiFi and Home Assistant API configured
- [x] RF receiver configured on GPIO2 (dump all protocols)
- [x] OTA updates configured

### In Progress

- [ ] Flash firmware to XIAO ESP32C6 via USB
- [ ] Verify device connects to WiFi and appears in Home Assistant

### Planned

- [ ] Capture RF codes from Levolor remote (using `remote_receiver` with `dump: all`)
- [ ] Identify the RF protocol and codes for Up / Down / Stop
- [ ] Enable `remote_transmitter` on GPIO3 with captured codes
- [ ] Create Home Assistant cover entity with open/close/stop controls
- [ ] Test full round-trip: Home Assistant → ESP32 → RF → Levolor blinds

## File Organization

```
RF-Blind-Controller/
├── CLAUDE.md                 # This file - AI assistant guidance
├── README.md                 # Project documentation for users
├── esphome/
│   └── levolor-blind-controller.yaml  # ESPHome device configuration
└── docs/
    └── rf-codes.md           # Captured RF codes and protocol notes (future)
```

## Important Notes for AI Assistants

### Framework Constraint

The ESP32-C6 does **not** support the Arduino framework in ESPHome. Always use `type: esp-idf` in the framework configuration. This is a common mistake — never suggest `type: arduino` for this board.

### ESPHome Secrets

- Use `!secret` references in YAML configs, never hardcode credentials
- The secrets file (`secrets.yaml`) lives on the Raspberry Pi, not in this repo
- Editable via the "SECRETS" button in the ESPHome dashboard

### RF Code Capture Workflow

1. Flash firmware with `remote_receiver` and `dump: all`
2. Open ESPHome logs (dashboard → Logs)
3. Press buttons on the physical Levolor remote near the receiver
4. Capture the protocol type and code values from the logs
5. Add corresponding `remote_transmitter` entries to the YAML

### Board Naming

The user repurposed this board from a previous "waveshare-esp32c6" config. References to `waveshare-esp32c6.yaml` in ESPHome refer to the same device. The canonical name is `levolor-blind-controller`.

### General Guidelines

- Always read existing files before modifying them
- Keep ESPHome YAML clean and well-commented
- Document pin assignments and RF protocol details as they are discovered
- Update this CLAUDE.md as the project evolves
- Be mindful of ESP32-C6 memory constraints (327KB RAM, 1.8MB Flash)
- Test YAML changes by compiling before flashing
