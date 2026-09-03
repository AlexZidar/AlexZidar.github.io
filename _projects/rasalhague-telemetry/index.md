---
layout: post
title: "Rasalhague: Long-Range Telemetry Flight Computer"
description: A custom-designed, high-power long-range telemetry flight computer engineered for Swamp Launch Student Rocketry. Powered by an ESP32-S3 microcontroller, Rasalhague integrates a 2W (33 dBm) LoRa transceiver, u-blox SAM-M10Q multi-constellation GNSS, BMP581 precision barometer, LSM6DSOX 6-axis IMU, and MMC5983MA electronic compass. It features dynamic RF power/spreading factor staging, high-speed 4GB onboard eMMC black-box logging, and an active-cooled RF power amplifier stage.
skills: 
  - C++
  - PCB Design
  - KiCad
  - Embedded Systems
  - Avionics
  - Python
  - FreeRTOS

main-image: /back_full.png
card-image: /rasalhague_card.png
order: 4
---

---
## Complete Electrical Schematic:
{% include image-gallery.html images="schematic.png" height="600" %}  

---

## Overview:
**Rasalhague** is a high-performance, long-range telemetry flight computer designed for high-powered rocketry with **Swamp Launch Student Rocketry**. Built to operate both as an onboard rocket transmitter and as a ground station receiver, Rasalhague solves the critical challenge of telemetry loss during extreme altitude flights and long-distance drifts.

By combining an ESP32-S3 microcontroller with a 2W (33 dBm) Semtech SX1262 LoRa front-end, multi-sensor avionics array, and vibration-immune 4GB eMMC storage, Rasalhague streams flight telemetry in real time while maintaining a complete, high-frequency flight record.

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

To streamline launch-day operations, I created a custom PyQt6 configuration application. Through a simple USB-C interface, team members can:
- Configure amateur radio callsign (FCC Part 97 identification)
- Set LoRa carrier frequency, bandwidth, and power profiles
- Define flight-event acceleration thresholds and descent rate triggers
- Review live pinout references and diagnostic status before arming on the pad

---

## 2W LoRa RF Subsystem & Dynamic Staging:
Rasalhague pairs a Semtech SX1262 transceiver with an EBYTE E22-900M33S power amplifier module capable of transmitting up to **+33 dBm (2000 mW)** on 915 MHz. To optimize link budget and battery life, the firmware implements dynamic RF flight staging:
- **PAD (0):** Transmits at low power (10 dBm, SF7) to avoid saturating nearby ground station receivers on the launch rail.
- **BOOST (1) & COAST (2):** Upon detecting launch acceleration (>3.5g), the board throttles to full 2W power and SF8 to beam high-rate telemetry through vehicle burnout and apogee.
- **DESCENT (3) & RECOVERY (4):** At parachute deployment, the radio switches to maximum spreading factor (SF11) at 2W power, dedicating bandwidth to high-reliability GPS recovery packets to guarantee recovery even in dense brush or distant terrain.

---

## Sensor Array & Black-Box eMMC Storage:
All flight-critical sensors interface over a dedicated 400 kHz I2C bus:
- **Bosch BMP581:** High-precision barometric pressure sensor for sub-meter altitude resolution.
- **STMicroelectronics LSM6DSOX:** 6-DOF IMU operating at 833 Hz ODR (16g accelerometer, 2000 dps gyroscope).
- **Memsic MMC5983MA:** 100 Hz 3-axis electronic compass for attitude determination.
- **u-blox SAM-M10Q:** Multi-constellation GNSS receiver streaming binary UBX NAV-PVT packets at 25 Hz.
- **4GB eMMC Storage:** Replaces vibration-prone physical microSD card slots with high-speed surface-mount eMMC memory operated over a 1-bit SD_MMC bus.
- **Active Thermal Management:** Integrated 5V fan driver on Pin 34 keeps the RF power amplifier cool during extended high-power transmissions.

---

## Firmware Flight State Machine Snippet:
### Dynamic RF power and modulation control based on sensor triggers:
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
- **Power Architecture:** BQ24074 Li-Ion Charger & Power Path, 4.8A 5.25V Boost, 3.3V LDO
