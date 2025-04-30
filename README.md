# Smart Kitchen Hood Upgrade

This project shows how I upgraded my traditional kitchen hood into a **Smart Kitchen Hood** using ESP32, ESPHome, and Home Assistant.

---

## How It Works

Firstly, you have to understand how your kitchen hood works.  
Start by inspecting the **controller board PCB**.

- The controller has **3 relays** for controlling the motor power.
- The **LED light** is powered by **12V DC** directly from the board.
- By tracing the wiring, I found that:
  - **Relays** transmit **240V AC** directly from the power supply to the motor.
  - Changing the fan speed **switches different relays**.
  - Using a **test pen**, I unplugged the motor wiring and observed:
    - It always **starts with the Low Speed relay active for 3 seconds** before switching to the desired relay for the selected speed.

---

## About the Button Board

- The original button panel is a **remote board** with **5 touch buttons**:
  - Power
  - Speed Low
  - Speed Medium
  - Speed High
  - Light
- The remote board communicates with the controller via **UART**.

**Important Discovery**:
- UART only allows **single master to single slave** communication.
- Trying to "share" the UART between the original controller and ESP failed.
- **Solution**: I replaced the entire controller board with my own ESP-based system.

---

## New Design

- Replaced the controller board with an **ESP32 and 4-channel relay module**.
- For the light:
  - Installed a **240V AC to 12V DC converter**.
  - Controlled the 12V output using the **4th relay**.
- Replaced the touch button board with **5x HTTM capacitive touch sensors**.
- ![ESP button](button.avif){ width=40% }
- Connected everything to the ESP32 board.

---

## Hardware List

| Item | Description |
|:----|:------------|
| ESP32 Development Board | Main microcontroller |
| 4-Channel Relay Module | To switch fan speeds and light |
| 5x HTTM Capacitive Touch Sensors | To replace the original touch buttons |
| AC to DC Power Supply (240V AC to 12V DC) | To power the kitchen hood light |
| Jumper Wires | For connecting components |
| Custom wiring harness | For clean and safe wiring |
| Screw Terminals / Connectors | For safer AC wiring connections |
| Enclosure Box (Optional) | To house the ESP32 and relays safely |

---

## Wiring Diagram (Overview)

- **ESP32 GPIOs** connected to:
  - Relay IN1, IN2, IN3, IN4 (for Fan Low, Medium, High, and Light control)
  - HTTM Touch Sensors (for manual fan and light control)
- **Relay outputs**:
  - Fan motor 240V AC power lines.
  - Light 12V DC power line (through the AC/DC converter).

*(Wiring diagram drawing: coming soon)*

---

## Software Configuration (ESPHome)

### Logic

- **Relay Outputs**: Each relay is mapped to a motor speed or light control.
- **Binary Light**: A simple ON/OFF control for the light.
- **Template Fan**:
  - 3 fan speeds (Low, Medium, High).
  - When turning ON the fan, it always activates the **Low relay first for 3 seconds**, then switches to the desired speed.
- **Template Number**:
  - A `number` component is used to **sync fan speed control** between Home Assistant and ESP.

---

## Features

- Manual control using **touch buttons**.
- Smart control via **Home Assistant** dashboard.
- Sync between **physical button presses** and **smart control**.
- **Automation-ready**: Can create automations based on cooking time, temperature, or humidity.

---

## Wiring Diagram
![ESP Board](espboard.avif)

## ESPHome Configuration
See `kitchen-hood.yaml` for full details.

## How it works
- Fan speed is controlled using a Number entity (1-3)
- Relays are toggled based on the selected speed
- Fully synchronized with Home Assistant controls
  

## Example ESPHome YAML

```yaml
output:
  - platform: gpio
    id: relay1
    pin: GPIO13
  - platform: gpio
    id: relay2
    pin: GPIO14
  - platform: gpio
    id: relay3
    pin: GPIO12
  - platform: gpio
    id: relay4
    pin: GPIO16  # Light relay

light:
  - platform: binary
    id: fan_light
    name: "Kitchen Hood Light"
    output: relay4

fan:
  - platform: template
    name: "Kitchen Hood Fan"
    id: hood_fan
    speed_count: 3
    on_turn_on:
      then:
        - script.execute:
            id: set_fan_speed
            speed_level: !lambda 'return id(kitchen_hood_speed).state;'
    on_turn_off:
      then:
        - output.turn_off: relay1
        - output.turn_off: relay2
        - output.turn_off: relay3
    on_speed_set:
      then:
        - script.execute:
            id: set_fan_speed
            speed_level: !lambda 'return (int)x;'

number:
  - platform: template
    name: "Kitchen Hood Speed"
    id: kitchen_hood_speed
    min_value: 1
    max_value: 3
    step: 1
    restore_value: true
    optimistic: true
    on_value:
      then:
        - script.execute:
            id: set_fan_speed
            speed_level: !lambda 'return (int)x;'

script:
  - id: set_fan_speed
    parameters:
      speed_level: int
    then:
      - output.turn_off: relay1
      - output.turn_off: relay2
      - output.turn_off: relay3
      - if:
          condition:
            lambda: 'return speed_level == 1;'
          then:
            - output.turn_on: relay2
      - if:
          condition:
            lambda: 'return speed_level == 2;'
          then:
            - output.turn_on: relay2
            - delay: 3s
            - output.turn_on: relay1
            - output.turn_off: relay2
      - if:
          condition:
            lambda: 'return speed_level == 3;'
          then:
            - output.turn_on: relay2
            - delay: 3s
            - output.turn_on: relay3
            - output.turn_off: relay2
