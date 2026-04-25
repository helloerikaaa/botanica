#  botanica

> A self-sufficient smart greenhouse ecosystem. ESP32-powered sensors, automated watering, climate control — built from the ground up with MicroPython.

![Status](https://img.shields.io/badge/status-phase%201-4ade80?style=flat-square)
![Platform](https://img.shields.io/badge/platform-ESP32-blue?style=flat-square)
![Language](https://img.shields.io/badge/language-MicroPython-yellow?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

##  Roadmap

| Phase | Status | Description |
|-------|--------|-------------|
| **1 — Sensors** |  In progress | Soil moisture monitoring + web dashboard |
| **2 — Environment** |  Planned | Temperature, humidity & light sensors |
| **3 — Watering** |  Planned | Automated watering by threshold or schedule |
| **4 — Climate** |  Planned | Fan, humidifier & heater control |
| **5 — Lighting** |  Planned | Grow lights with day/night cycles |
| **6 — Greenhouse** |  Planned | Full physical structure + integrated system |

---

##  Project Structure

```
botanica/

 README.md

 devices/                        # MicroPython code for each ESP32
    1/
       main.py                 # Node 1 — plants 1–5 + central dashboard
    2/
        main.py                 # Node 2 — plants 6–10, exposes /data

 config/
    plants.json                 # Plant names, pins, thresholds per device

 docs/
    wiring.md                   # Wiring diagrams per phase
    calibration.md              # How to calibrate moisture sensors
    roadmap.md                  # Detailed roadmap with hardware list

 hardware/
     phase1_bom.md               # Bill of materials — Phase 1
```

---

##  Getting Started

### Requirements

- 2× ESP32 (any variant)
- Resistive soil moisture sensors (one per plant)
- MicroPython firmware flashed on both ESP32s
- [Thonny IDE](https://thonny.org) or `mpremote` to upload code

### 1. Flash MicroPython

Download the latest MicroPython firmware for ESP32 from [micropython.org](https://micropython.org/download/ESP32_GENERIC/) and flash it:

```bash
esptool.py --port /dev/ttyUSB0 erase_flash
esptool.py --port /dev/ttyUSB0 write_flash -z 0x1000 esp32-micropython.bin
```

### 2. Configure

Edit the top section of each `main.py`:

```python
WIFI_SSID     = "your_wifi"
WIFI_PASSWORD = "your_password"

PLANTAS = [
    ("Plant name",  32),   # (name, ADC pin)
    ...
]

DRY_VALUE = 4095   # Raw ADC value in dry soil  — calibrate first
WET_VALUE = 1500   # Raw ADC value in wet soil  — calibrate first
```

### 3. Upload

**Start with ESP32 #2:**
```bash
mpremote connect /dev/ttyUSB0 cp devices/esp32_2/main.py :main.py
```
Note the IP shown in the serial monitor.

**Then ESP32 #1** — set `IP_ESP32_2` to the IP from above:
```bash
mpremote connect /dev/ttyUSB1 cp devices/esp32_1/main.py :main.py
```

### 4. Open Dashboard

Open a browser on any device connected to the same WiFi:
```
http://<ESP32_1_IP>
```

---

##  Wiring

### Moisture Sensor → ESP32

```
Sensor VCC  →  3.3V
Sensor GND  →  GND
Sensor AOUT →  GPIO 32 (or 33, 34, 35, 36, 39)
```

>  Only use ADC1 pins (32–39) — ADC2 pins conflict with WiFi.

### Pin Map (Phase 1)

| Plant | ESP32 | Pin |
|-------|-------|-----|
| 1–5   | #1    | 32, 33, 34, 35, 36 |
| 6–10  | #2    | 32, 33, 34, 35, 36 |

---

##  Calibration

Accurate readings require calibrating your specific sensors and soil:

1. Insert sensor in **completely dry soil** → note the raw value → set as `DRY_VALUE`
2. Water thoroughly and insert sensor → note raw value → set as `WET_VALUE`

See [docs/calibration.md](docs/calibration.md) for detailed instructions.

---

##  Dashboard

The self-hosted dashboard runs directly on ESP32 #1 and is accessible from any browser on your local network. It shows:

- Live moisture percentage per plant
- Status badge (Needs water / Dry / Ideal / Wet)
- 8-hour sparkline history per plant
- Summary of how many plants need watering

Data endpoint available at `http://<ESP32_IP>/data` (JSON).

---

##  Bill of Materials — Phase 1

| Component | Qty | Notes |
|-----------|-----|-------|
| ESP32 (any variant) | 2 | Dev board recommended |
| Resistive moisture sensor | 10 | Capacitive recommended for longevity |
| Jumper wires | — | Female-to-male |
| USB power supply | 2 | 5V 1A minimum |

---

##  Contributing

This project is personal but PRs and ideas are welcome. Open an issue to discuss new phases or hardware suggestions.

---

##  License

MIT — do whatever you want, just don't blame me if your plants die.
