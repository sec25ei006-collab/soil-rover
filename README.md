# Autonomous Soil-Profiling 4WD Rover (ESP32)

A compact, low-cost 4WD rover that drives to selected locations, measures **NPK, pH, electrical conductivity,
soil moisture and temperature** with a 7-in-1 RS485 soil sensor, tags each reading with **GPS** coordinates and
sends it over Wi-Fi (MQTT) to a dashboard for soil-health mapping and fertilizer planning.

> Goal: an affordable prototype supporting precision farming and sustainable resource use (SDG 2, 6, 12).

## Features
- ESP32 firmware: Modbus-RTU soil sensing, GPS tagging, MQTT telemetry, remote drive commands
- CRC-checked sensor frames, JSON telemetry schema, threshold-based zone flagging
- Host-side Python tools + unit tests (no hardware needed), synthetic sample dataset
- Optional extension: PWM-driven pump/solenoid for targeted spraying (see docs)

## Repository layout
```
firmware/rover/     ESP32 Arduino sketch + config.example.h
tools/              soil_utils.py (decode/validate/flag), simulate.py (sample data generator)
tests/              unit tests (python -m unittest)
data/               sample_readings.csv, sample_payload.json
docs/               TECHNICAL_DESIGN.md (architecture diagrams, wiring, data schema)
LICENSE             MIT
```

## Hardware (bill of materials)
| Part | Notes |
|---|---|
| ESP32 DevKit (30/38-pin) | main controller, Wi-Fi |
| 7-in-1 RS485 soil sensor (moisture, temp, EC, pH, N, P, K) | Modbus RTU, 4800 baud, address 1 (default) |
| MAX485 TTL-to-RS485 module | DE/RE tied to GPIO4 |
| NEO-6M (or similar) GPS | UART, 9600 baud |
| L298N / dual motor driver + 4 DC gear motors, 4WD chassis | |
| 12 V Li-ion/LiPo pack + buck converter (5 V) | sensor needs 5-30 V per its datasheet |

Wiring: see `docs/TECHNICAL_DESIGN.md`.

## Setup
1. **Software**: install Arduino IDE 2.x, add the ESP32 board package, and install libraries
   `TinyGPSPlus`, `PubSubClient`, `ArduinoJson` (v7).
2. **Configuration (environment variables)**: copy `firmware/rover/config.example.h` to
   `firmware/rover/secrets.h` (git-ignored) and set:

   | Variable | Meaning | Example |
   |---|---|---|
   | `WIFI_SSID` / `WIFI_PASS` | Wi-Fi credentials | |
   | `MQTT_HOST` / `MQTT_PORT` | MQTT broker | `broker.hivemq.com` / `1883` |
   | `MQTT_USER` / `MQTT_PASS` | broker credentials (optional) | |
   | `DEVICE_ID` | unique rover name | `rover-01` |
   | `MQTT_TOPIC` | telemetry topic | `soilrover/rover-01/telemetry` |
   | `SAMPLE_INTERVAL_MS` | time between samples | `5000` |
3. Select board **ESP32 Dev Module**, open `firmware/rover/rover.ino`, upload, and open Serial Monitor at 115200.
4. Subscribe to `soilrover/rover-01/#` with any MQTT client (e.g. MQTT Explorer) to see telemetry.
5. Drive the rover by publishing `{"cmd":"fwd"}` / `left` / `right` / `stop` to `soilrover/rover-01/cmd`.

## Run the host tools and tests (no hardware)
```bash
python3 tools/simulate.py            # regenerates data/sample_readings.csv
python3 -m unittest discover -s tests -v
```

## Telemetry format
```json
{"device":"rover-01","ts":1790000000,"lat":12.9716,"lon":77.5946,"moisture":35.0,"temperature":25.5,
 "ec":480.0,"ph":6.6,"n":40.0,"p":25.0,"k":120.0,"flags":[]}
```
Units: moisture %, temperature C, EC uS/cm, N/P/K mg/kg. `flags` lists out-of-range parameters.

## Calibration and limitations
- Low-cost NPK sensors give *indicative* values; calibrate against lab tests before agronomic decisions.
- Verify the Modbus register map and scaling against your sensor's datasheet (`tools/soil_utils.py`).
- Wireless range and GPS accuracy (~2.5 m typical) limit mapping resolution.

## License
MIT - see [LICENSE](LICENSE).
