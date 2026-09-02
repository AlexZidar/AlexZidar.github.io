---
layout: post
title: "Marfik: High-Speed Flight DAQ & Acoustic Recorder"
description: An ultra-high-speed avionics flight data acquisition system and static test instrumentation board engineered for Swamp Launch Student Rocketry, part of the Ophiuchus Constellation series. Featuring an ESP32-S3 dual-core microcontroller, Marfik records 500 Hz deterministic multi-sensor telemetry (6-DoF dynamics, 400G impact accel, high-temp thermocouple, dual pressure transducers, and 4-channel 24-bit strain gauges) alongside 16 kHz I2S acoustic audio to onboard 4GB parallel eMMC flash storage with post-flight Wi-Fi recovery.

skills: 
  - C++
  - FreeRTOS
  - PCB Design
  - KiCad
  - Embedded Systems
  - Avionics
  - Python
  - Signal Processing

main-image: /marfik_schematic.png
---

---

## Complete Electrical Schematic:
{% include image-gallery.html images="marfik_schematic.png" height="400" %}  

---

## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/marfik-daq/Marfik.glb" alt="Marfik Flight DAQ 3D CAD Model" height="480" %}

---

## Overview:
**Marfik** is a dual-core, high-speed avionics flight computer and static test data acquisition system developed for **Swamp Launch Student Rocketry** as part of the **Ophiuchus Constellation** avionics architecture. 

Engineered around the Espressif ESP32-S3 microcontroller operating at 240 MHz, Marfik delivers 500 Hz multi-sensor deterministic telemetry logging and 16 kHz acoustic audio recording directly to onboard 4 GB eMMC flash storage. Designed for high-G aerospace environments, Marfik eliminates mechanical vibration vulnerabilities while enabling rapid wireless data offload over Wi-Fi after recovery.

---

## High-Density Multi-Sensor Array:
Marfik integrates an extensive suite of embedded and analog front-end sensors designed to capture rapid vehicle dynamics and extreme physical environments:

- **Primary Dynamics (6-DoF):** STMicroelectronics LSM6DSO32TR 3-axis accelerometer (±16 G) and 3-axis gyroscope (±2000°/s) sampled at 500 Hz.
- **Extreme-G Impact & Stage Separation:** High-G accelerometer (H3LIS331DLTR) with a ±400 G range (12-bit resolution) to capture violent motor ignition transients, pyrotechnic staging shocks, and ground impact events.
- **3-Axis Magnetometer:** Memsic MMC5983MA 18-bit electronic compass with 0.0625 mG field resolution.
- **Atmospheric & Environmental Telemetry:** Bosch BME690 measuring pressure (300–1100 hPa), temperature, relative humidity, and gas resistance.
- **High-Temperature Thermocouple Digitizer:** Microchip MCP9601 / MCP9600 cold-junction compensated thermocouple amplifier (-200°C to +1370°C with 0.0625°C resolution, supporting Types K, J, T, N, S, E, B, R) for rocket casing and exhaust boundary monitoring.
- **Dual Pressure Transducers:** TI ADS1015 12-bit differential ADC capturing dual 0–3.3V or 0–5V analog pressure transducers at 100 Hz.
- **4-Channel Strain Gauge Load Cells:** Dual Nuvoton NAU7802 24-bit ADCs (128x PGA) digitizing 4 full-bridge strain gauge channels at 500 Hz for airframe aerodynamic strain and static motor thrust.

---

## 16 kHz I2S Acoustic Audio Recording:
To capture combustion instability, transonic acoustic resonance, and staging events, Marfik includes an onboard **DMM-4026-B-R** digital MEMS microphone. The audio is streamed directly into an internal FreeRTOS circular DMA ring buffer over I2S at 16 kHz (16-bit mono PCM) and automatically finalized with standard WAV audio headers onto the storage partition.

---

## High-Speed 4GB eMMC & Dual Redundant Storage:
- **Primary eMMC Storage:** Micron 4GB eMMC flash memory configured in a 4-bit parallel `SD_MMC` bus, sustaining over 40 MB/s write throughput. Because the flash memory is soldered directly to the PCB, it is completely immune to the connector chatter and unseating issues that plague physical microSD cards under high launch vibration.
- **Secondary MicroSD Mirror:** Secondary SPI MicroSD card slot providing automated post-flight mirrored copies of binary telemetry files (`.bin`), configuration manifests (`.json`), and acoustic tracks (`.wav`).

---

## Standalone Ground Control Desktop GUI:
{% include image-gallery.html images="marfik_gui.png" height="380" %}

I developed a Python PyQt6 desktop application for pre-flight calibration, live USB diagnostics, and post-flight recovery:
- **Pre-Flight Configuration:** Set launch acceleration triggers, burnout detection windows, sensor offsets, and strain gauge calibration factors.
- **Wireless SoftAP Offload:** After touchdown, Marfik hosts an autonomous Wi-Fi SoftAP (`Marfik-[BoardName]`) and embedded HTTP web server, allowing field recovery teams to download complete flight data logs directly to a laptop or smartphone without disassembling the rocket airframe.

---

## Firmware Architecture:
The firmware is architected using PlatformIO with FreeRTOS multitasking:
```C++
// Sensor Acquisition RTOS Task (Deterministic 500 Hz Execution)
void vSensorTask(void *pvParameters) {
    TickType_t xLastWakeTime = xTaskGetTickCount();
    const TickType_t xFrequency = pdMS_TO_TICKS(2); // 2ms = 500 Hz
    
    for (;;) {
        TelemetryFrame frame;
        frame.timestamp_us = esp_timer_get_time();
        
        // Synchronous read of 6-DoF IMU and High-G Accelerometer
        readIMU(frame.accel_x, frame.accel_y, frame.accel_z, frame.gyro_x, frame.gyro_y, frame.gyro_z);
        readHighG(frame.impact_accel_x, frame.impact_accel_y, frame.impact_accel_z);
        
        // Check state machine flight triggers
        updateFlightState(frame);
        
        // Push to high-speed lockless eMMC write queue
        xQueueSend(xStorageQueue, &frame, 0);
        
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
    }
}
```

---

## Technical Specifications:
- **MCU:** Espressif ESP32-S3 Dual-Core 240 MHz (FreeRTOS)
- **Flash Storage:** 4 GB Parallel 4-bit eMMC Flash (~40 MB/s) + MicroSD Slot
- **Acoustic Audio:** 16 kHz 16-Bit Mono I2S MEMS Microphone (DMM-4026-B-R)
- **Dynamics Sensors:** ST LSM6DSO32TR (6-DoF, 500 Hz) & H3LIS331DLTR (400G Shock)
- **Analog Front Ends:** Dual NAU7802 (4x 24-bit strain gauges) & TI ADS1015 (2x pressure transducers)
- **Thermal Digitizer:** Microchip MCP9601 Cold-Junction Thermocouple Digitizer
- **Display & Audio:** SSD1306 128x32 OLED Display + Piezo Acoustic Locator Beacon
- **Recovery Wireless:** Standalone 802.11 b/g/n Wi-Fi Access Point & HTTP File Server
