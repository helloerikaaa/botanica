#  Sensor Calibration Guide

Resistive moisture sensors are not factory-calibrated — each one behaves slightly differently depending on the sensor, your soil mix, and your pot size. This guide walks you through getting accurate readings.

---

## Why Calibration Matters

The sensor outputs a raw ADC value between 0 and 4095. Without calibration:
- `4095` could mean dry **or** disconnected
- `0` could mean soaking wet **or** short circuit

By measuring your actual dry and wet values, botanica maps the raw reading to a 0–100% scale that makes sense for your specific setup.

---

## Tools You Need

- ESP32 with MicroPython flashed
- Your moisture sensor connected to an ADC pin
- A cup of water
- The pot with your actual soil mix

---

## Step 1 — Read Raw Values

Upload this temporary calibration script to your ESP32:

```python
from machine import ADC, Pin
import time

PIN = 32  # Change to your sensor pin

adc = ADC(Pin(PIN))
adc.atten(ADC.ATTN_11DB)

while True:
    readings = [adc.read() for _ in range(10)]
    avg = sum(readings) // len(readings)
    print(f"Raw: {avg}")
    time.sleep(1)
```

Open the serial monitor in Thonny (or `mpremote connect /dev/ttyUSB0 repl`).

---

## Step 2 — Get DRY_VALUE

1. Fill your pot with your actual soil mix
2. Let it dry completely (leave it 2–3 days without watering)
3. Insert the sensor into the dry soil
4. Wait 10 seconds for readings to stabilize
5. Note the average value → this is your **`DRY_VALUE`**

> Typical range: **3500–4095**

---

## Step 3 — Get WET_VALUE

1. Water the pot thoroughly until water drains from the bottom
2. Wait 5 minutes for the water to distribute evenly
3. Insert the sensor into the wet soil
4. Wait 10 seconds for readings to stabilize
5. Note the average value → this is your **`WET_VALUE`**

> Typical range: **1000–2000**

---

## Step 4 — Update Your Config

In `main.py` for each ESP32, update the values per plant:

```python
PLANTAS = [
    ("Sedum burrito", 32),   # pin 32
    ...
]

DRY_VALUE = 3800   # your measured value
WET_VALUE = 1200   # your measured value
```

Or if you want per-plant calibration in the future, edit `config/plants.json`:

```json
{ "slot": 1, "name": "Sedum burrito", "pin": 32, "dry": 3800, "wet": 1200 }
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Always reads 0% | Sensor not reaching soil / disconnected | Check wiring |
| Always reads 100% | Short circuit / sensor submerged | Remove from water |
| Reads 100% when dry | DRY_VALUE too low | Re-measure with fully dry soil |
| Jumpy readings | Electrical noise | Average more samples (increase from 5 to 20) |
| Sensor oxidizing fast | Resistive sensor left in wet soil | Use capacitive sensors for phase 2 |

---

## Capacitive vs Resistive

| | Resistive | Capacitive |
|--|-----------|------------|
| Cost | Very cheap | Slightly more expensive |
| Accuracy | Good | Better |
| Longevity | Oxidizes over time | Lasts years |
| Wiring | Same | Same |

For succulents (soil is dry most of the time), resistive sensors last much longer than with tropical plants. For phase 2 of botanica, switching to capacitive is recommended.

---

## Recalibrating

Recalibrate if you:
- Change your soil mix
- Move to a different pot size
- Notice readings drifting over weeks
- Replace a sensor