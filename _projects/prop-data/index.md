---
layout: post
title: "Serpens: Next-Generation Propulsion DAQ"
description: The next-generation propulsion data acquisition system for Swamp Launch Student Rocketry, part of the Ophiuchus Constellation series of custom avionics boards. Advancing upon the original DAQ V1, Serpens integrates an ESP32-S3 microcontroller, 4-channel 0-4.5V pressure transducer digitizers, dual high-precision load cell bridges, 3 hardware-monitored thermocouple interfaces, and a 10-channel addressable ARGB diagnostic ring with high-speed SD logging and wireless live telemetry streaming.

skills: 
  - C++
  - Python
  - PCB Design
  - KiCad
  - Embedded Systems
  - Propulsion
  - Data Analytics
  - Control Systems

main-image: /daqlaunch.jpg
---

---

## Continuation of Original DAQ Project:
Serpens represents the next-generation evolution of our static test stand instrumentation, directly succeeding and advancing the capabilities of the [Original Propulsion DAQ (V1)](/archive/prop-data/index/) now located in the Archive. While V1 established our initial baseline for high-speed hot-fire recording, Serpens was completely redesigned from the ground up as part of the **Ophiuchus Constellation** family of aerospace electronics.

---

## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/prop-data/Serpens.glb" alt="Serpens Propulsion DAQ 3D CAD Model" height="480" %}

---

## Hot Fire Testing & Motor Verification:
{% include image-gallery.html images="daqlaunch.jpg" height="400" %}  

---

## Overview:
**Serpens** is a high-speed, multi-channel propulsion data acquisition and test stand controller designed for solid rocket motor and hybrid propulsion hot-fire testing with **Swamp Launch Student Rocketry**. 

Engineered around the Espressif ESP32-S3 dual-core microcontroller running at 240 MHz, Serpens provides high-frequency synchronous sampling of thrust, multiple combustion chamber pressures, feed line pressures, and high-temperature thermal profiles. The system logs raw data to an onboard high-speed SPI microSD card while simultaneously transmitting live telemetry wirelessly to our ground control software over Wi-Fi.

---

## Sensor Array & Subsystems:
- **4-Channel Pressure Transducers:** Texas Instruments ADS1015IDGS at `0x48` configured to digitize 0–4.5V industrial pressure transducers across all 4 channels (chamber pressure, pre/post injector pressures, and manifold pressures).
- **Dual Strain Gauge Load Cells:** Nuvoton NAU7802 24-bit differential bridge ADC at `0x2A` utilizing Channels A & B to capture multi-axis thrust profiles and side loads with 128x programmable gain.
- **3 Dedicated Thermocouple Channels:** Three Maxim MAX31856 precision thermocouple digitizers on a dedicated secondary SPI bus (`SPI_1`: MISO 40, CLK 41, MOSI 42) with independent chip selects (CS 4, 5, 6) and hardware data-ready interrupts (DRDY 14, 15, 16) for real-time monitoring of nozzle throat erosion and casing temperatures up to +1370°C.
- **10-Channel ARGB Status Ring (WS2812B-1010):** A custom chain of 10 micro addressable RGB LEDs positioned next to each terminal block providing real-time hardware diagnostics:
  - **Solid Green:** Channel configured, sensor online, healthy baseline.
  - **Solid Red:** Channel disabled / unconfigured in current test profile.
  - **Flashing Red:** Hardware fault, open-circuit thermocouple, or ADC out-of-range alert.
- **Local Display & Range Safety:** 0.91-inch SSD1306 OLED (`0x3C`) for local status readouts, passive buzzer, and high-power indicator lamp for audible and visual hot-fire countdown warnings.
- **Battery Fuel Gauge:** Precision voltage divider (499 kΩ / 100 kΩ) on Pin 7 for accurate battery pack health monitoring.

---

## Serpens Ground Control GUI:
{% include image-gallery.html images="serpens_gui.png" height="380" %}

I developed a Python desktop application using PyQt6 and pyqtgraph to communicate with Serpens over local Wi-Fi telemetry:
- **Real-Time Visualization:** Simultaneous real-time waveform plotting of 4 pressure transducers (PSI), dual load cells (lbf), and 3 thermocouple channels (°C).
- **Test Stand Arming & Countdown:** Remote countdown sequencer, ignition relay arming, and synchronized high-speed recording triggers.
- **Live Terminal Health:** Mirrored ARGB channel status display matching the physical board's LED indicators.
- **Sensor Calibration Suite:** Multi-point calibration wizards for load cells and pressure transducers with real-time tare and offset compensation.

---

## Automated Motor Ballistics & Analysis Reports:
{% include image-gallery.html images="serpens_ballistics_report.png,serpens_thrust_plots.png" height="420" %}

Following each test, the integrated analysis engine processes raw test data to automatically compute motor ballistics and generate comprehensive PDF qualification reports:
- **Total Impulse (It):** Numerical integration of the thrust-time curve (∫ F dt).
- **Maximum & Average Thrust (F_max, F_avg)**
- **Burn & Action Times (tb, ta):** Standard 5% and 10% threshold definitions.
- **Chamber Pressure Integral (∫ Pc dt):** Characteristic velocity (C*) and thrust coefficient (CF).
- **Motor Classification:** Instant classification according to Tripoli Rocketry Association / NAR standards (e.g. K737).

---

## Hardware Architecture:
- **Processor:** Espressif ESP32-S3 Dual-Core Xtensa LX7 @ 240 MHz
- **Pressure ADC:** TI ADS1015 (4-Channel 12-Bit Differential, I2C `0x48`)
- **Thrust ADC:** Nuvoton NAU7802 (Dual-Channel 24-Bit Differential, I2C `0x2A`)
- **Thermal Digitizers:** 3x Maxim MAX31856 (K-Type, SPI_1 Bus)
- **Local Diagnostics:** 10x WS2812B-1010 ARGB Ring + SSD1306 128x32 OLED
- **Storage:** High-Speed MicroSD Card Slot (Dedicated SPI Bus)
- **Communications:** 802.11 b/g/n Wi-Fi Telemetry & USB-C Programming
