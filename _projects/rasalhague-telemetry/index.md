---
layout: post
title: "Rasalhague: Long-Range Telemetry Flight Computer"
description: A custom long-range telemetry flight computer built for Swamp Launch Student Rocketry. Powered by an ESP32-S3, Rasalhague pairs a 2W (33 dBm) LoRa transceiver with GPS, a precision barometer, IMU, and electronic compass. It features dynamic RF power staging, 4GB onboard eMMC storage for vibration-resistant flight logging, and active cooling for the RF stage.
skills: 
  - Avionics
  - KiCad Design
  - PCB Design
  - Product Development
  - Embedded Systems
  - FreeRTOS
  - C++
  - Python

main-image: /back_full.png
card-image: /rasalhague_card.png
order: 2
---

---
## Complete Electrical Schematic:
{% include image-gallery.html images="schematic.png" height="600" %}  

---

## Overview:
Rasalhague is a long-range telemetry flight computer built for high-power rocketry with Swamp Launch Student Rocketry. Designed to serve as both an onboard rocket transmitter and a ground station receiver, it ensures reliable data transfer during high-altitude flights and long-distance drifts. 

Using an ESP32-S3 microcontroller paired with a 2W Semtech SX1262 LoRa module, onboard sensors, and 4GB of eMMC storage, Rasalhague streams flight telemetry in real time while logging high-rate sensor data locally.

---

## Fabricated Circuit Boards:
{% include image-gallery.html images="fab_front.jpg" height="420" %}
{% include image-gallery.html images="fab_back.jpg" height="420" %}

---

## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/rasalhague-telemetry/Rasalhague.glb" alt="Rasalhague Telemetry Flight Computer 3D Model" height="480" %}

---

## Pre-Flight Configuration Tool:
{% include image-gallery.html images="config_gui.png" height="380" %}
{% include image-gallery.html images="config_gui_pinout.png" height="380" %}

To make launch-day operations straightforward, I built a desktop configuration app in Python using PyQt6. Over USB-C, operators can:
- Configure amateur radio callsign for FCC Part 97 identification
- Set LoRa frequency, bandwidth, and power profiles
- Set launch acceleration triggers and descent rate thresholds
- Check pinouts and run sensor diagnostics before arming on the pad

---

## 2W LoRa RF Subsystem & Dynamic Staging:
Rasalhague pairs a Semtech SX1262 transceiver with an EBYTE E22-900M33S power amplifier module capable of transmitting up to 33 dBm (2W) on 915 MHz. To balance signal range and battery life, the firmware shifts radio output and modulation across flight phases:
- **Pad (0):** Transmits at low power (10 dBm, SF7) to avoid swamping nearby ground station receivers on the launch rail.
- **Boost (1) & Coast (2):** Once launch acceleration (>3.5g) is detected, the board steps up to 2W power and SF8 to stream high-rate telemetry through burnout and apogee.
- **Descent (3) & Recovery (4):** At parachute deployment, the radio shifts to a higher spreading factor (SF11) at 2W, maximizing link margin for GPS recovery packets while the vehicle descends and lands in brush or uneven terrain.

---

## Sensor Array & eMMC Storage:
Onboard sensors communicate over a dedicated 400 kHz I2C bus:
- **Bosch BMP581:** High-accuracy barometric pressure sensor for altitude calculation.
- **STMicroelectronics LSM6DSOX:** 6-DOF IMU (±16g accelerometer, ±2000°/s gyro) sampled at 833 Hz.
- **Memsic MMC5983MA:** 100 Hz 3-axis electronic compass for attitude tracking.
- **u-blox SAM-M10Q:** Multi-constellation GNSS receiver streaming binary UBX packets at 25 Hz.
- **4GB eMMC Storage:** Surface-mount eMMC flash memory over a 1-bit SD_MMC bus, avoiding the card unseating issues microSD slots can face under rocket vibration.
- **Active Cooling:** Integrated 5V fan driver on Pin 34 to keep the RF amplifier cool during long high-power transmissions.

---

## Firmware Flight State Machine Snippet:
### Dynamic LoRa power scaling based on flight state:
```C++
// Dynamic LoRa power scaling based on flight state
void updateRadioStaging(FlightState currentState) {
    switch (currentState) {
        case STATE_PAD:
            // Low power on pad to prevent receiver saturation
            radio.setOutputPower(10); // 10 dBm (10 mW)
            radio.setSpreadingFactor(7);
            break;

        case STATE_BOOST:
        case STATE_COAST:
            // Maximum power during high-speed ascent and apogee coast
            setPAActiveCoolingFan(true);
            radio.setOutputPower(33); // 33 dBm (2000 mW / 2W)
            radio.setSpreadingFactor(8);
            break;

        case STATE_DESCENT:
        case STATE_LANDED:
            // Maximum range recovery mode: high SF prioritizes GPS link margin
            radio.setOutputPower(33);
            radio.setSpreadingFactor(11);
            break;
    }
}
```

---

## Hardware Specifications:
- **MCU:** Espressif ESP32-S3-N8 (Dual-Core Xtensa LX7 @ 240 MHz, FreeRTOS)
- **RF Transceiver:** EBYTE E22-900M33S (Semtech SX1262 + 2W PA, 915 MHz)
- **GNSS Receiver:** u-blox SAM-M10Q-00B (GPS / Galileo / GLONASS / BeiDou @ 25 Hz)
- **Barometer:** Bosch BMP581 (High-accuracy ±0.5 Pa)
- **IMU:** ST LSM6DSOX (16g Accel, 2000 dps Gyro @ 833 Hz)
- **Magnetometer:** Memsic MMC5983MA 3-Axis Electronic Compass
- **Onboard Storage:** 4GB eMMC Flash Memory (1-Bit SD_MMC)
- **Status Display:** 0.91-inch 128x32 OLED Display (SSD1306)
- **Power System:** BQ24074 Li-Ion Charger & Power Path, 5.25V Boost, 3.3V LDO

