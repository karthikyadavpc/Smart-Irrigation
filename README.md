# Smart Irrigation System - ESP32 IoT Sensor Front-End

![System architecture](images/system_architecture.png)


## What it does

A capacitive soil sensor feeds a conditioned analog signal into ESP32 ADC1 GPIO34. The firmware samples, median-filters and smooths the reading, then maps two locally measured endpoints into a **relative moisture index**. A hysteresis controller requests bounded watering pulses with a soak interval, tank interlock, sample-age checks and a cumulative session on-time limit. The ESP32 hosts a read-only local Wi-Fi dashboard.

**Default build:** pump output disabled; an LED shows the watering request. Calibration and manual USB arming are required even for the LED demonstration. An optional build supports the documented low-voltage MOSFET pump driver after hardware verification. No mains wiring is included.

## Features

- Classic ESP32 DevKit / Arduino framework, pinned PlatformIO environment.
- GPIO34 on ADC1 for sensing while Wi-Fi is active.
- Nine-read median every 500 ms, followed by EMA smoothing (alpha 0.25).
- Dry/wet captures over 20 frames; minimum endpoint separation and stability checks.
- Calibration saved as one packed item in ESP32 Preferences/NVS.
- Example 35% start / 55% stop hysteresis, 5 s pulse, 60 s soak, 15 s session budget.
- Startup disarmed; tank and sensor faults turn the requested output off and latch.
- Read-only local AP dashboard with live JSON and a rolling chart, no cloud dependency.
- Portable C++ tests, synthetic replay, build logs, diagrams and editable report.

These are demonstration settings, not crop-specific agronomic recommendations. The index is **not volumetric water content**. Two-point mapping does not characterize full sensor or ADC linearity.

## Quick start

1. Read `hardware/WIRING.md` and connect the sensor, front-end, LED and tank jumper/switch.
2. Open this folder in VS Code with PlatformIO.
3. Build/upload the default `esp32dev` environment; monitor at 115200 baud.
4. Use `dry`, `wet`, `save` commands as explained in `docs/CALIBRATION.md`.
5. Obtain a fresh valid reading, confirm tank-ready, then send `arm` over serial.
6. Connect to the Wi-Fi name/password printed on USB serial. Open the displayed local URL.
7. Use `disarm` to stop and clear the session. The physical pump remains disabled by default.

```bash
python -m pip install platformio==6.1.18
pio run -e esp32dev
pio run -e esp32dev --target upload
pio device monitor --baud 115200
```

If editing the dashboard, run `python tools/embed_dashboard.py` before rebuilding. No additional third-party firmware libraries are required.

## Bench pin map

| ESP32 pin | Connection |
|---|---|
| 3V3 / GND | Compatible capacitive sensor supply and common reference |
| GPIO34 | Conditioned analog sensor output (ADC1 input) |
| GPIO33 | Tank-ready switch to GND; internal pull-up, open means not ready |
| GPIO25 | 330-ohm series resistor and indicator LED to GND |
| GPIO26 | Optional MOSFET gate interface; disabled by default |

Use a classic ESP32-compatible board, not ESP32-C3/S3 pin assignments. GPIO34 has no internal pull-up/down. Keep ADC input in the documented measurement range; do not connect a 5 V signal directly.

## Local dashboard

Open `web/dashboard.html` directly on your computer for an explicitly labeled **synthetic preview**. Live mode is served by the ESP32 and reads `/api/status` every two seconds. No internet or external JavaScript service is required. The AP password is generated each boot and printed locally.

There are no HTTP control endpoints: calibrating, saving, arming and disarming use USB serial. The dashboard is local HTTP, not a cloud service or authenticated internet application. Do not port-forward it.

## Calibration and control

- No active calibration is assumed at first boot; both endpoints must be measured.
- Endpoints can increase or decrease with moisture; both polarities are supported.
- ADC rail values and an endpoint span under 200 counts are rejected.
- Faults require manual `disarm`, correction, fresh readings and `arm`.
- The LED follows the watering request even when physical pump mode is disabled.
- A tank jumper is acceptable only for an attended LED bench demo, not pump use.
- The session budget is cumulative requested on-time, not a measured volume or flow limit.

## Verification

Run the native software checks with Python 3 and g++ (C++17):

```bash
python tests/run_tests.py
```

**Executed:** 46 native checks, 6 dashboard logic checks with mocked responses, the synthetic replay and both ESP32 cross-builds passed. Real-browser layout was not verified because its engine download failed. See `reports/VALIDATION.md`, native test logs and firmware build logs for scope and limits. Synthetic data demonstrates logic only. No water savings, moisture accuracy, crop yield or physical pump reliability is claimed.

## Package map

| Folder | Contents |
|---|---|
| `src/`, `include/`, `lib/` | Firmware, configuration and portable controller |
| `web/` | Dashboard source with offline synthetic preview |
| `hardware/` | Interconnect notes, component list and measurement templates |
| `images/` | Editable SVG/PNG technical diagrams and any labeled UI preview |
| `tests/`, `data/` | Native checks and synthetic fixtures |
| `docs/` | Setup, calibration, algorithm, interview and GitHub guides |
| `reports/` | PDF/Markdown report, test logs and hardware-results template |
| `photographs/` | Shot list for authentic prototype photographs |
| `tools/` | Report, diagram, dashboard and calibration-analysis tools |
| `.github/workflows/` | Native tests and both ESP32 compile checks |

## Evidence and authorship

The analog conditioner, GPIO allocation, controller, local dashboard and optional driver are reference design choices newly prepared for this package. A physical analog comparator, cloud backend, PCB fabrication, original prototype recovery and field testing are not implemented. The source report in the resume is not proof that these particular files were previously built or measured.

Add your actual photographs, board revision, calibration results and contribution history. Review and understand the generated reference code before presenting it. See `LICENSE-NOTE.md` before selecting a license or adding team-owned material.
