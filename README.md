# Smart Irrigation System: IoT Sensor Front-End (ESP32)

An ESP32-based soil-moisture sensing front-end with a focus on **signal conditioning**: voltage dividers, comparators and ADC interfacing, plus calibration of sensor **linearity and offset errors**. The firmware turns the conditioned signal into a moisture index and controls a pump with hysteresis and safety limits.

<!-- TODO: add a photo of your setup: images/setup.jpg -->
![Setup](images/setup.jpg)

## Project status

Tick these off as you verify them on your own hardware, and delete any item you did not do before publishing.

- [✔] Signal-conditioning circuit built and checked with a multimeter / oscilloscope
- [✔] Two-point calibration performed on the real sensor
- [✔] Multi-point linearity and offset characterization done (`tools/analyze_calibration.py`)
- [✔] Comparator thresholds measured 
- [✔] Pump control verified with an LED / dummy load before connecting a real pump


## Features
- Soil-moisture signal conditioning: voltage divider, RC filter and comparator stage in front of the ESP32 ADC
- ADC1 pin with 16x oversampling and calibrated millivolt readings (`analogReadMilliVolts`)
- Two-point calibration stored in flash, with offset and linearity characterization script for multi-point data
- Hysteresis pump control (separate on/off thresholds) with minimum off time, maximum run time and cooldown
- Fail-safe: a disconnected or shorted sensor, or an invalid calibration, forces the pump off
- Dry-run mode (default): prints pump decisions without driving any output
- Logic separated from hardware (`moisture_logic.h`) and unit-tested on a PC

## Signal chain

```
 Soil sensor ──► Voltage divider ──► RC low-pass ──┬──► ESP32 ADC1 (GPIO34)
 (analog out)    (scale to ≤ ~3 V)   (noise filter) │
                                                    └──► Comparator (hysteresis) ──► ESP32 GPIO (optional)

 ESP32 GPIO26 ──► MOSFET / relay module ──► Pump (separate supply, common ground)
```

See [`hardware/README.md`](hardware/README.md) for design equations and [`hardware/BOM.md`](hardware/BOM.md) for the parts list.

## Hardware

| Component | Purpose | Your part |
|---|---|---|
| ESP32 DevKit | Controller, 12-bit ADC | [Node MCU] |
| Soil-moisture sensor (analog output) | Moisture measurement | [YL-69] |
| Resistor divider | Scales sensor output into the ADC range | [R1 = 10 kΩ, R2 = 20 kΩ ] |
| RC filter | Noise reduction at the ADC pin | [R = 10 kΩ, C = 0.1 µF (100nF)] |
| Comparator | Hardware threshold with hysteresis | [LM393 (Dual Differential Comparator)] |
| MOSFET or relay module | Switches the pump | [IRLB8721 (Logic-level N-Channel MOSFET)] |
| Pump and supply | Watering | [5V or 12V DC Submersible Water Pump + matching external power adapter] |

## Repository structure

```
smart-irrigation-esp32/
├── firmware/
│   ├── src/main.cpp              # ESP32 application
│   ├── include/config.h          # pins, safety and pump settings
│   ├── include/moisture_logic.h  # calibration and pump logic (hardware independent)
│   ├── test/host/test_logic.cpp  # unit tests that run on a PC
│   └── platformio.ini
├── hardware/                     # design notes, BOM, schematic files
├── docs/
│   ├── calibration.md            # calibration and characterization procedure
│   └── test_plan.md              # tests and results
├── tools/
│   ├── analyze_calibration.py    # offset / gain / linearity from calibration points
│   └── example_SYNTHETIC.csv     # made-up data to demonstrate the script only
├── images/
├── LICENSE
└── README.md
```

## Getting started

1. Install [PlatformIO](https://platformio.org/) and open the `firmware/` folder.
2. Check the pins in `firmware/include/config.h` against your wiring. `DRY_RUN` is `true` by default.
3. Build and upload to the ESP32, then open the serial monitor at 115200 baud.
4. Calibrate: put the sensor in the dry reference condition and send `d`, then the wet reference and send `w`.
5. Send `a` to arm. Watch the CSV output and the printed pump decisions.
6. Only after the electronics are verified with an LED or dummy load, set `DRY_RUN` to `false`.

Serial commands: `d` dry reference, `w` wet reference, `c` clear calibration, `a` arm, `x` disarm, `s` status.

CSV columns: `t_ms, pin_mv, moisture_pct, comparator, pump_cmd, armed, fault`

Run the logic tests on a PC (no ESP32 needed):

```bash
g++ -std=c++17 -I firmware/include firmware/test/host/test_logic.cpp -o test_logic && ./test_logic
```

## Calibration and characterization

Two-point calibration runs on the device. For offset and linearity analysis, record several known moisture points and run:

```bash
python3 tools/analyze_calibration.py your_data.csv
```

The procedure, the definitions of offset error and nonlinearity, and a data template are in [`docs/calibration.md`](docs/calibration.md).

## Results

Fill in with your own measurements. Describe how each was obtained in [`docs/test_plan.md`](docs/test_plan.md).

| Metric | Measured | Test conditions |
|---|---|---|
| Offset (at reference = 0) | [3150 mV ] | [Open Air, 25°C] |
| Gain | [-25.0mV] | [Sandy Loam, 25°C, 3.3V Supply] |
| Nonlinearity|  |[±3.5% of full scale] | [5 multi-point gravimetric points (0%–50% VWC)] |
| Repeatability  | [±4.2 mV std. deviation] | [N = 20, 10-second intervals] |
| Comparator trip points (rising / falling) | [1900 mV / 2200 mV] | [Saturated threshold / Dry trigger] |

> **Note:** The pump thresholds in `config.h` are starting values and are not validated for any particular soil or plant. The moisture index is relative to your calibration references and is not a measured volumetric water content.

## What I learned
- Designing a soil-moisture signal-conditioning front-end with voltage dividers, comparators and ADC interfacing
- Calibrating sensor linearity and offset errors
- Separating hardware-independent logic so it can be tested on a PC

## Possible improvements
- WiFi/MQTT reporting to make it a full IoT node
- Temperature compensation of the sensor reading
- Deep-sleep operation for battery power

## Author
**Karthik Yadav P C**, ECE graduate, Bengaluru
[LinkedIn](https://linkedin.com/in/karthik-yadav-p-c-0335b6264) · Karthikyadavpc@gmail.com

## License
Released under the MIT License. See [LICENSE](LICENSE).
