#  Botanica — Detailed Roadmap

Full plan for building a self-sufficient smart greenhouse from scratch.

---

## Phase 1 — Soil Moisture Monitoring  In Progress

**Goal:** Know when each plant needs water.

### Features
- Read soil moisture from 10 resistive sensors
- Self-hosted web dashboard (no cloud, no app)
- 24-hour history per plant
- Live status: Needs water / Dry / Ideal / Wet
- Distributed architecture: 2× ESP32, 5 plants each
- Central dashboard on ESP32 #1 aggregates all 10 plants

### Hardware
| Component | Qty |
|-----------|-----|
| ESP32 dev board | 2 |
| Resistive soil moisture sensor | 10 |
| Micro-USB cable | 2 |
| 5V 1A USB power supply | 2 |
| Jumper wires (F-M) | ~30 |
| Small breadboard | 2 |

### Code
- `devices/esp32_1/main.py`
- `devices/esp32_2/main.py`
- `config/plants.json`

---

## Phase 2 — Environment Sensing  Planned

**Goal:** Know the temperature, humidity and light inside the greenhouse.

### Features
- Temperature & humidity per node (DHT22 or AHT20)
- Light level sensor (BH1750)
- Display all values on the dashboard alongside moisture
- Log data for trend analysis

### Hardware
| Component | Qty | Notes |
|-----------|-----|-------|
| DHT22 sensor | 2 | One per ESP32 node |
| BH1750 light sensor | 1–2 | I2C, very accurate |
| 10kΩ resistor | 2 | Pull-up for DHT22 data pin |

### Code changes
- Add DHT22 and BH1750 reading to `main.py`
- Extend `/data` JSON with `temperature`, `humidity`, `lux`
- Update dashboard to show environment cards

---

## Phase 3 — Automated Watering  Planned

**Goal:** Water plants automatically when soil moisture drops below a threshold.

### Features
- One small pump per plant (or per group)
- Trigger watering when moisture < threshold (configurable per plant)
- Safety: max watering duration to avoid flooding
- Manual override from dashboard
- Watering log (when, how long, before/after moisture)

### Hardware
| Component | Qty | Notes |
|-----------|-----|-------|
| 5V mini submersible pump | 10 | One per plant |
| 5-channel relay module | 2 | One per ESP32 |
| Silicone tubing (4mm) | ~5m | |
| Water reservoir | 1–2 | Depends on layout |
| 5V 3A power supply | 2 | For pumps (separate from ESP32) |

### Code changes
- Add relay control to `main.py`
- Watering logic: check moisture → trigger pump → log event
- Dashboard: manual water button + watering history

---

## Phase 4 — Climate Control  Planned

**Goal:** Maintain ideal temperature and humidity inside the greenhouse automatically.

### Features
- Fan triggered when temperature exceeds threshold
- Humidifier triggered when humidity drops too low
- Optional: small heater for cold nights
- All thresholds configurable from dashboard

### Hardware
| Component | Qty | Notes |
|-----------|-----|-------|
| 12V PC fan (80mm or 120mm) | 1–2 | |
| USB ultrasonic humidifier | 1 | |
| Relay module (additional channels) | 1 | |
| 12V power supply | 1 | For fan |
| Optional: ceramic PTC heater | 1 | Low wattage |

### Code changes
- Climate control loop in `main.py`
- Hysteresis logic to avoid rapid on/off cycling
- Dashboard: climate controls + history charts

---

## Phase 5 — Grow Lighting  Planned

**Goal:** Supplement or replace natural light with programmable grow lights.

### Features
- LED grow light strips (full spectrum)
- Configurable day/night schedule
- Gradual sunrise/sunset dimming via PWM
- Override from dashboard

### Hardware
| Component | Qty | Notes |
|-----------|-----|-------|
| Full-spectrum LED strip (12V) | 1–2m | |
| IRLZ44N MOSFET | 1–2 | Logic-level, for PWM dimming |
| 12V 2A power supply | 1 | |
| Aluminum channel for LED | optional | Better heat dissipation |

### Code changes
- PWM control via `machine.PWM`
- Schedule: on/off times + dim level
- Sunrise/sunset ramp function

---

## Phase 6 — Full Greenhouse  Planned

**Goal:** Put everything together in a real physical structure.

### Features
- All previous phases integrated
- Physical enclosure (wood, PVC, or aluminum frame + polycarbonate panels)
- Centralized wiring with terminal blocks
- Single power entry point
- Optional: battery backup for ESP32s

### Structure Options
| Option | Cost | Difficulty |
|--------|------|------------|
| PVC pipe frame + plastic sheeting | Low | Easy |
| Aluminum extrusion + polycarbonate | Medium | Medium |
| Wood frame + greenhouse film | Low | Easy |

### Final Dashboard
- All 10 plants
- Temperature, humidity, light
- Watering history
- Climate status
- Light schedule
- Alerts for anything out of range

---

## Hardware Total Estimate (All Phases)

| Phase | Estimated Cost (USD) |
|-------|----------------------|
| 1 — Sensors | ~$15–25 |
| 2 — Environment | ~$10–15 |
| 3 — Watering | ~$30–50 |
| 4 — Climate | ~$20–35 |
| 5 — Lighting | ~$15–25 |
| 6 — Structure | ~$30–80 |
| **Total** | **~$120–230** |

> Prices vary significantly depending on your supplier (AliExpress vs local).

---

## Architecture Evolution

```
Phase 1                Phase 3                Phase 6
                              
ESP32 #1               ESP32 #1               ESP32 #1
   5 sensors    →       5 sensors    →       5 sensors
ESP32 #2                    5 pumps              5 pumps
   5 sensors           ESP32 #2               ESP32 #2
                             5 sensors            5 sensors
                             5 pumps              5 pumps
                          DHT22 + BH1750         DHT22 + BH1750
                                               Fan + Humidifier
                                               Grow lights
                                               Physical structure
```