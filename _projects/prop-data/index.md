---
layout: post
title: "Serpens: Next-Generation Propulsion DAQ"
description: A multi-channel propulsion data acquisition board built for Swamp Launch Student Rocketry to support solid and hybrid motor static testing. Succeeding our first DAQ, Serpens uses an ESP32-S3 to read 4 pressure transducer channels, dual load cells, and 3 thermocouple inputs, featuring a 10-channel LED diagnostic ring, local SD logging, and wireless telemetry.

skills: 
  - Data Analytics
  - KiCad Design
  - PCB Design
  - Product Development
  - Propulsion
  - Control Systems
  - Embedded Systems
  - C++
  - Python

main-image: /daqlaunch.jpg
order: 4
---

---

## Continuation of Original DAQ Project:
Serpens succeeds our [Original Propulsion DAQ (V1)](/archive/prop-data/index/), which is now in the archive. While V1 served as our initial test stand logger, Serpens was rebuilt from scratch to expand our sensor channels, improve noise immunity, and integrate into our broader rocket electronics stack.

---

## Complete Electrical Schematic:
{% include image-gallery.html images="serpens_schematic.png" height="600" %}  

---

## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/prop-data/Serpens.glb" alt="Serpens Propulsion DAQ 3D CAD Model" height="480" %}

---

## Hot Fire Testing & Motor Verification:
{% include image-gallery.html images="daqlaunch.jpg" height="400" %}  

---

## Overview:
Serpens is a multi-channel data acquisition and test stand controller built for solid and hybrid rocket motor testing with Swamp Launch Student Rocketry.

Powered by an ESP32-S3 running at 240 MHz, the board synchronously samples motor thrust, chamber and feed line pressures, and exhaust temperatures. It writes high-speed raw logs to a microSD card while streaming real-time telemetry over Wi-Fi to our test control station.

---

## Sensor Array & Subsystems:
- **4-Channel Pressure Transducers:** TI ADS1015 12-bit ADC reading 0 to 4.5V industrial transducers for chamber pressure, injector feed lines, and manifold pressures.
- **Dual Load Cell Inputs:** Nuvoton NAU7802 24-bit differential ADC reading load cells with up to 128x programmable gain for thrust and side loads.
- **3 Dedicated Thermocouple Channels:** Three Maxim MAX31856 digitizers on a secondary SPI bus for monitoring nozzle throat erosion and motor casing temperatures up to 1370°C.
- **10-Channel ARGB Status Ring:** Addressable LEDs placed next to each screw terminal to give immediate visual feedback on sensor connections:
  - **Solid Green:** Sensor online and reading within expected baseline.
  - **Solid Red:** Channel unassigned in the current test profile.
  - **Flashing Red:** Sensor fault, open thermocouple wire, or out-of-range signal.
- **Local Display & Safety Indicators:** Onboard 0.91-inch OLED display, piezo buzzer, and high-power indicator lamp for countdown warnings on the pad.
- **Battery Monitoring:** Resistor divider on an analog pin to track supply voltage during long test days.

---

## Ground Control GUI:
{% include image-gallery.html images="serpens_gui.png" height="380" %}

I developed a Python desktop application with PyQt6 and pyqtgraph to display live test data over Wi-Fi:
- **Live Graphing:** Real-time plots for pressure curves in psi, thrust in lbf, and temperatures in °C.
- **Countdown & Arming:** Remote test countdown sequencer, ignition relay arming, and synchronized recording triggers.
- **Status Mirroring:** Graphical view mirroring the board's LED indicators so operators can verify all sensors are connected from the bunker.
- **Sensor Calibration:** Calibration tools for load cells and pressure transducers with quick tare and zeroing.

---

## Motor Ballistics & Analysis Reports:
{% include image-gallery.html images="serpens_ballistics_report.png,serpens_thrust_plots.png" height="420" %}

After each burn, a Python analysis script processes the raw log files to calculate standard propulsion metrics and generate a test summary:
- **Total Impulse:** Numerical integration of the thrust curve over burn duration.
- **Peak and Average Thrust:** Max and average thrust values throughout the burn.
- **Burn and Action Time:** Standard 5% and 10% threshold definitions.
- **Chamber Pressure Integral:** Characteristic velocity and thrust coefficient.
- **Motor Classification:** Automatic Tripoli / NAR letter classification, such as a K-class motor.

---

## Hardware Specifications:
- **Processor:** Espressif ESP32-S3 Dual-Core Xtensa LX7 @ 240 MHz
- **Pressure ADC:** TI ADS1015 4-channel 12-bit differential ADC at I2C `0x48`
- **Thrust ADC:** Nuvoton NAU7802 dual-channel 24-bit differential ADC at I2C `0x2A`
- **Thermal Digitizers:** 3x Maxim MAX31856 K-type digitizers on secondary SPI bus
- **Local Diagnostics:** 10x WS2812B-1010 ARGB Ring + SSD1306 128x32 OLED
- **Storage:** High-speed microSD card slot over a dedicated SPI bus
- **Communications:** 802.11 b/g/n Wi-Fi Telemetry & USB-C Programming
