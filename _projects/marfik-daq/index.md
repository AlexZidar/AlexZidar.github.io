---
layout: post
title: "Marfik: High-Speed Flight DAQ & Acoustic Recorder"
description: A high-speed flight data acquisition and static test instrumentation board designed for Swamp Launch Student Rocketry. Built around an ESP32-S3, Marfik logs 500 Hz telemetry across 6-DoF dynamics, a 400G impact accelerometer, thermocouples, dual pressure transducers, and 4-channel strain gauges alongside 16 kHz audio directly to onboard 4GB eMMC flash, with wireless data download over Wi-Fi after landing.

skills: 
  - Avionics
  - KiCad Design
  - PCB Design
  - Product Development
  - Data Analytics
  - Signal Processing
  - Embedded Systems
  - FreeRTOS
  - C++
  - Python

main-image: /marfik_cover.jpg
order: 3
---

---

## Complete Electrical Schematic:
{% include image-gallery.html images="marfik_schematic.png" height="600" %}  

---

## Fabricated & Assembled Hardware:
{% include image-gallery.html images="front_OLED.jpg, back.jpg" height="420" %}

---

## Interactive 3D CAD Model:
{% include model-viewer.html model="/_projects/marfik-daq/Marfik.glb" alt="Marfik Flight DAQ 3D CAD Model" height="480" %}

---

## Overview:
Marfik is a high-speed avionics and test stand data acquisition board developed for Swamp Launch Student Rocketry as part of our custom electronics stack.

Built around an ESP32-S3 running at 240 MHz, Marfik logs multi-sensor telemetry at 500 Hz alongside 16 kHz audio directly to onboard 4GB eMMC flash memory. Because the storage is soldered directly to the board, it holds up under high vibrations that could shake a microSD card loose, while built-in Wi-Fi lets us offload data in the field without opening the avionics bay.

---

## Sensor Suite:
Marfik integrates a sensor suite designed to capture fast vehicle dynamics, motor performance, and structural strain:

- **6-DoF IMU:** STMicroelectronics LSM6DSO32TR (±16g accelerometer and ±2000°/s gyroscope) sampled at 500 Hz.
- **High-G Impact Accelerometer:** H3LIS331DLTR with a ±400g range to measure motor ignition shocks, staging events, and landing impacts.
- **3-Axis Magnetometer:** Memsic MMC5983MA electronic compass for orientation data.
- **Environmental Sensor:** Bosch BME690 measuring ambient pressure from 300 to 1100 hPa, temperature, and humidity.
- **Thermocouple Interface:** Microchip MCP9601 amplifier with cold-junction compensation for monitoring motor casing and exhaust temperatures up to 1370°C.
- **Dual Pressure Transducers:** TI ADS1015 12-bit ADC reading two analog pressure sensors at 100 Hz.
- **4-Channel Strain Gauges:** Dual Nuvoton NAU7802 24-bit ADCs reading four full-bridge strain gauge channels at 500 Hz for static thrust stands or airframe strain.

---

## 16 kHz I2S Audio Recording:
To capture motor sound, staging noises, and airframe vibrations during flight, Marfik includes an onboard DMM-4026-B-R digital MEMS microphone. Audio streams over I2S into a FreeRTOS DMA ring buffer at 16 kHz in 16-bit mono and is saved directly as standard WAV files on the flash storage.

---

## Onboard 4GB eMMC & Dual Storage:
- **Primary eMMC Storage:** Micron 4GB eMMC flash memory running over a 4-bit parallel SD_MMC bus, sustaining up to 40 MB/s write speeds. Soldering the memory directly to the PCB eliminates card unseating and contact chatter during high-G launch and landing.
- **Secondary MicroSD Mirror:** MicroSD card slot providing an optional backup copy with binary telemetry files (.bin), JSON configuration manifests, and WAV audio tracks.

---

## Ground Control Desktop GUI:
{% include image-gallery.html images="marfik_gui.png" height="380" %}

I developed a Python PyQt6 desktop app for bench diagnostics, calibration, and data offload:
- **Pre-Flight Setup:** Set acceleration triggers, sensor zero-offsets, and strain gauge calibration constants over USB.
- **Wireless Data Download:** After recovery, Marfik can host a local Wi-Fi access point and web server, letting us download flight logs directly to a phone or laptop in the field without taking apart the airframe.

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
- **Flash Storage:** 4 GB Parallel 4-bit eMMC Flash and MicroSD Slot
- **Acoustic Audio:** DMM-4026-B-R MEMS Microphone over 16 kHz I2S
- **Dynamics Sensors:** ST LSM6DSO32TR 6-DoF IMU and H3LIS331DLTR 400G Shock Accelerometer
- **Analog Front Ends:** Dual NAU7802 ADCs for 4 strain gauge channels and TI ADS1015 for 2 pressure transducers
- **Thermal Digitizer:** Microchip MCP9601 Cold-Junction Thermocouple Digitizer
- **Display & Audio:** SSD1306 128x32 OLED Display + Piezo Acoustic Locator Beacon
- **Recovery Wireless:** Standalone 802.11 b/g/n Wi-Fi Access Point & HTTP File Server
