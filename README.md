# ESPHome Garage Door

[Slovenská verzia](README-sk.md)

ESPHome package for safe control of a garage door or another motorized gate through Home Assistant. The project uses three relay inputs on the gate operator (`OPEN`, `CLOSE`, `STOP`) and a physical fully-closed sensor as the main source of truth.

## What It Does

- Controls the gate through a Home Assistant `cover` entity.
- Uses separate pulses for `OPEN`, `CLOSE`, and `STOP`, with relay interlock protection against simultaneous activation.
- Starts safely: all relays are turned off on boot and the gate state is synchronized from the CLOSED sensor.
- Supports WireGuard connectivity so the ESPHome device can stay reachable in the same secure network as Home Assistant.
- Handles gates without an upper end-stop sensor: the open state is assumed after a configured delay, while the closed state must be confirmed by the physical sensor.
- Tracks opening statistics: current open time, last open duration, total open time, open count, and CLOSE command count.
- Exposes diagnostic entities for WireGuard, gate logic state, restart, and statistics reset.

## Files

- `Garage-Door.yml` - the main ESPHome package with the gate logic.
- `esphome-sample.yml` - a minimal device configuration that loads the package from GitHub.
- `secrets.yaml.sample` - a template for Wi-Fi, API, OTA, WireGuard, board, and GPIO values.

## Requirements

- ESPHome 2026.8.1 or newer.
- An ESP32 board supported by ESPHome.
- Home Assistant with ESPHome Builder, or a local ESPHome installation.
- A gate operator with separate `OPEN`, `CLOSE`, and `STOP` inputs.
- A physical fully-closed sensor connected to a GPIO input.
- Valid WireGuard settings when using the current package version.

## Installation In Home Assistant

1. Open `Home Assistant > ESPHome Builder`.
2. Click `+ New Device`.
3. Choose `Continue`, then `Skip this step` or `Advanced setup`, depending on your ESPHome Builder version.
4. Create an empty device configuration.
5. Copy the contents of `esphome-sample.yml` into the new device configuration.
6. Fill your ESPHome `secrets.yaml` with values based on `secrets.yaml.sample`.
7. Check the GPIO pins, board, framework, and WireGuard settings.
8. Save the configuration and run `Install`.
9. After the first flash, verify the garage door entity in Home Assistant and physically test `OPEN`, `STOP`, and `CLOSE`.

## Secrets

At minimum, configure:

- device name and friendly name,
- board type and ESP32 framework,
- GPIO pins for `OPEN`, `CLOSE`, `STOP`, and the CLOSED sensor,
- Wi-Fi SSID, Wi-Fi password, and fallback AP password,
- ESPHome API key and OTA password,
- WireGuard address, device private key, peer public key, endpoint, port, and allowed IP ranges.

Use `secrets.yaml.sample` as a template. Do not commit the real `secrets.yaml` file.

## Gate State Logic

The CLOSED sensor has the highest priority. When it is active, the gate is treated as closed and any active movement sequence is stopped.

When the CLOSED sensor is inactive, the gate is treated as open or moving. Because the project does not use an upper end-stop sensor, the open state is marked as assumed after `assume_open_after`. During closing, the logic waits for physical confirmation from the CLOSED sensor; if it does not arrive before `close_timeout`, the logic enters a `CLOSE timeout` error state.

## Planned Improvements

- Support for using the project without WireGuard.
- Automatic pre-installation functional testing with ESP32 > ESP32.
- A colored beacon or status indicator for the gate.
- Easier support for I2C modules and basic extensions.

## Safety Notes

Before connecting the controller to a gate operator, verify the operator's electrical requirements and input type. Relay outputs from an ESP32 module must not directly switch voltage or current beyond the relay module rating. Run the first tests under supervision and keep a way to disconnect power immediately.
