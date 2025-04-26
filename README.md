# Smart Kitchen Hood (ESPHome)

This project converts a traditional kitchen hood into a smart fan with 3 speed levels and a light control using ESPHome.

## Features
- Control fan ON/OFF
- 3-speed levels (Low, Medium, High)
- Light control (ON/OFF)
- Full Home Assistant integration

## Hardware
- ESP8266 / ESP32 board
- 3 Relays for Fan speeds
- 1 Relay for Light
- Wiring to the kitchen hood

## Wiring Diagram
![ESP Board](espboard.avif)

## ESPHome Configuration
See `kitchen-hood.yaml` for full details.

## How it works
- Fan speed is controlled using a Number entity (1-3)
- Relays are toggled based on the selected speed
- Fully synchronized with Home Assistant controls
