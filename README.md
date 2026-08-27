# 2-Axis Gimbal (Arduino + ESP32)

A 2-axis (pitch/roll) stabilized gimbal built around an Arduino for real-time servo/IMU control and an ESP32 for wireless telemetry and remote control.

## Overview

This project stabilizes a mounted payload (camera, sensor, etc.) against tilt and roll disturbances using IMU feedback and closed-loop PID control on two servo-driven axes. The Arduino handles the tight control loop (IMU read → PID → servo write), while the ESP32 exposes a Wi-Fi interface for live tuning, telemetry logging, and manual override — communicating with the Arduino over UART/I2C.

## Features

- Real-time pitch and roll stabilization via IMU feedback
- PID control loop per axis with tunable gains
- Wireless control/monitoring dashboard served from the ESP32 (Wi-Fi AP or station mode)
- Manual override / setpoint control from a phone or browser
- Modular wiring — swap servos or IMU without rewriting control logic

## Hardware

| Component | Notes |
|---|---|
| Arduino (Uno/Nano or similar) | Runs the servo + IMU control loop |
| ESP32 Dev Board | Wi-Fi telemetry, web UI, remote setpoints |
| 2x SG90 (or MG90S) servos | Pitch and roll axes |
| IMU (e.g. GY-87 / MPU6050) | Orientation feedback, I2C |
| 3D-printed gimbal frame | Pan/tilt bracket + payload mount |
| 5V power supply (separate from logic rail) | Servos draw current spikes — don't power off USB alone |
| Jumper wires, breadboard or perfboard | Wiring |

## Wiring

**IMU → Arduino (I2C):**
- VCC → 5V (or 3.3V, check your IMU breakout)
- GND → GND
- SDA → A4 (Uno) / SDA pin
- SCL → A5 (Uno) / SCL pin

**Servos → Arduino:**
- Pitch servo signal → D9
- Roll servo signal → D10
- Servo power → external 5V rail (common ground with Arduino)

**Arduino ↔ ESP32 (UART):**
- Arduino TX → ESP32 RX (via voltage divider if Arduino is 5V logic)
- Arduino RX → ESP32 TX
- Common GND

> Adjust pin assignments in `config.h` to match your actual wiring.

## Software Setup

### Arduino
1. Install required libraries via Library Manager:
   - `Wire` (built-in)
   - `Adafruit MPU6050` / equivalent IMU library
   - `Servo`
2. Open `gimbal_control/gimbal_control.ino`
3. Set servo pins and PID gains in `config.h`
4. Upload to the Arduino

### ESP32
1. Install the ESP32 board package in Arduino IDE (Board Manager URL required — see [espressif/arduino-esp32](https://github.com/espressif/arduino-esp32))
2. Open `esp32_bridge/esp32_bridge.ino`
3. Set Wi-Fi credentials (or leave blank to boot as an AP) in `secrets.h`
4. Upload to the ESP32

## Usage

1. Power the servo rail and both boards.
2. On boot, the ESP32 either connects to your Wi-Fi network or starts its own AP (default: `Gimbal-Setup`, check serial monitor for IP).
3. Navigate to the ESP32's IP address in a browser to view live pitch/roll telemetry and adjust PID gains or setpoints.
4. The gimbal will actively correct pitch and roll as the base is tilted.

## Calibration

- Keep the gimbal level and stationary on boot — the IMU calibrates its zero offset during the first few seconds.
- If one axis oscillates, reduce that axis's `Kp` first, then reintroduce `Kd` to damp overshoot.
- Watch for I2C address conflicts if you add additional sensors.

## Troubleshooting

| Symptom | Likely Cause |
|---|---|
| Servo jitters constantly | PID gains too aggressive, or noisy IMU power rail |
| Axes cross-talk (one moves the other) | Mounting/orientation mismatch in axis mapping, not a wiring fault |
| ESP32 can't reach Arduino data | Check UART TX/RX are crossed, not parallel, and baud rates match |
| IMU not detected | Confirm I2C address and wiring; run an I2C scanner sketch |

## License

MIT — feel free to fork, modify, and build on this.
