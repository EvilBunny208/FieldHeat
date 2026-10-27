[README.md](https://github.com/user-attachments/files/33992194/README.md)
# FieldHeat

FieldHeat is a cheap heat-stress monitor for high school practice fields. A sensor on the field measures Wet Bulb Globe Temperature (WBGT). A web app shows coaches the FHSAA heat level, the practice rules for that level, and an AI forecast for the next 12 hours.

I built it for the 2026 Congressional App Challenge in Florida's 19th District.

## Why

Florida's Zachary Martin Act requires high schools to monitor heat stress at outdoor practices. The law is named for a Lee County football player who died of heat stroke after a summer workout in 2017. Commercial WBGT meters cost $300–$480. FieldHeat measures the same thing for about $25 in parts at scale, and its forecast lets coaches plan practice before conditions get dangerous.

A thermometer only measures air temperature. WBGT also accounts for humidity, sun and wind, and all four together decide whether the body can cool itself.

## How it works

```
Sensor (ESP32) ──Wi-Fi──> Supabase database ──> Web app (phone or computer)
                                                   │
                                Open-Meteo weather forecast
```

1. The sensor reads air temperature, humidity and black-globe temperature once a minute and posts them to Supabase.
2. The web app converts each reading to WBGT (the formulas are in `analysis/heat_math.py`), shows the FHSAA level and rules, and logs a reading every hour.
3. The AI field model compares the sensor's readings with a physics estimate built from the weather forecast. A ridge regression learns how this field differs because of its sun exposure, surface, wind and humidity, then corrects the forecast for the next 12 hours. I measure its accuracy with leave-one-out validation, on hours the model didn't train on.

## Features

- Live WBGT with the FHSAA level and practice rules
- 12-hour AI forecast with a likely range, plus an alert when the level is about to change
- Best time to practice in the next 30 hours
- Practice planner that shows peak heat, FHSAA time limits and required rest breaks for a planned session
- Hourly heat log for compliance records

## Hardware

| Part | Purpose |
| --- | --- |
| ESP32-S3 DevKitC-1 | Microcontroller with Wi-Fi |
| SHT31 or SHT45 | Air temperature and humidity, inside a 3D-printed radiation shield |
| DS18B20 waterproof probe | Globe temperature, centered in the globe |
| 4" copper float, painted flat black | Black globe |
| USB power bank | Power |
| IP65 junction box, tripod | Housing, mounted at 3.5 ft |

### Wiring

| Sensor pin | ESP32-S3 pin |
| --- | --- |
| SHT VIN | 3V3 |
| SHT GND | GND |
| SHT SDA | GPIO 8 |
| SHT SCL | GPIO 9 |
| DS18B20 VCC | 3V3 |
| DS18B20 GND | GND |
| DS18B20 DAT | GPIO 4, with 4.7kΩ pull-up to 3V3 |

## Repository

| Path | Contents |
| --- | --- |
| `web/index.html` | Web app (HTML, CSS, JavaScript) |
| `firmware/fieldheat_sensor.ino` | ESP32 firmware (Arduino C++) |
| `analysis/heat_math.py` | WBGT formulas (Python) |
| `database/schema.sql` | Supabase table and access policies |

## Setup

1. Create a Supabase project and run `database/schema.sql`.
2. Fill in your Wi-Fi details and the Supabase URL and anon key in the firmware, then upload it with Arduino IDE (board: ESP32S3 Dev Module; libraries: OneWire, DallasTemperature, Adafruit SHT31 or SHT4x).
3. Open `web/index.html`, go to Settings, and enter the Supabase URL and anon key. Until you do, the app shows labeled sample data.

## Languages and tools

C++ (Arduino), JavaScript, HTML/CSS, Python, SQL. Supabase, Open-Meteo, Arduino IDE.

## Credits

- [Open-Meteo](https://open-meteo.com) weather forecast API
- [Supabase](https://supabase.com) database
- Arduino libraries: OneWire, DallasTemperature, Adafruit SHT31 / SHT4x
- Stull (2011) wet-bulb formula; ISO 7726 globe temperature method
- FHSAA WBGT activity guidelines

## Safety note

FieldHeat doesn't replace a school's certified WBGT device or its athletic trainer. Use it alongside them.
