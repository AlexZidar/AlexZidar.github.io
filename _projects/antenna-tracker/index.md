---
layout: post
title: Autonomous Rocket Tracker & Ground Station
description: An autonomous 2-axis antenna tracker built for Swamp Launch Student Rocketry to track high-power rockets during ascent and recovery. By keeping high-gain directional antennas pointed at the rocket, the ground station maintains a steady telemetry link and live onboard video feed throughout the flight. It uses a duplicate Rasalhague telemetry board as its central receiver and motion controller.
skills: 
  - CAD Design
  - SolidWorks
  - Mechatronics
  - Robotics
  - Product Development
  - Control Systems
  - 3D Printing
  - Embedded Systems
  - C++
  - Python

main-image: /pan_tilt.gif
card-image: /pan_tilt_zoom.gif
order: 1
---

---

## Overview:
During high-power rocket flights with Swamp Launch Student Rocketry, maintaining a reliable radio link is critical for real-time telemetry and live onboard video. Standard omnidirectional antennas struggle with path loss and signal fade as the vehicle climbs and drifts downrange. 

To solve this, I designed and built an autonomous 2-axis ground antenna tracker. The system points high-gain directional antennas at the rocket throughout its flight, keeping a strong signal margin and steady data stream from launch to touchdown.

---

## Interactive 3D Ground Station CAD Model:
{% include model-viewer.html model="/_projects/antenna-tracker/Project_Sauron.glb" alt="Project Sauron: Autonomous Ground Station Gimbal CAD" height="520" pitch="-90" %}

---

## Dual Board Architecture & Power:
The ground tracker and rocket avionics operate as a matched pair. The ground station uses a duplicate [Rasalhague Telemetry Flight Computer](/projects/rasalhague-telemetry/index/) as its receiver and motion controller:
- **RF Reception:** The onboard Semtech SX1262 LoRa module receives binary telemetry packets broadcast from the rocket at 915 MHz.
- **Local Reference:** The ground board uses its onboard GPS, BMP581 barometer, and electronic compass to establish a local East-North-Up (ENU) coordinate frame without manual calibration.
- **Dual Link:** The tracker carries high-gain directional antennas for both the 915 MHz LoRa telemetry stream and the 5.8 GHz digital video link.
- **Power System:** The ground board runs on a 1S Li-Ion battery, while motor power is delivered through a custom PCB utilizing USB-C Power Delivery (PD) connected to an external power bank.

---

## Live-Streamed Video Integration:
{% include image-gallery.html images="runcam_wifilink.png" height="380" %}

To stream video from the airframe, the rocket carries a RunCam WiFiLink digital transmitter in the avionics bay. The ground station's motorized gimbal keeps high-gain helical antennas aimed at the vehicle, maintaining video frame rates and image quality that would otherwise drop out as the rocket pulls away.

---

## Gimbal & Motion Control:
- **Actuation:** 2x NEMA 17 stepper motors driving the azimuth and elevation axes.
- **Closed-Loop Drivers:** Makerbase MKS SERVO42C closed-loop stepper controllers communicate directly with the ESP32-S3 over a hardware UART bus at 115200 baud.
- **Position Feedback:** Magnetic encoders monitor shaft positions at 20 kHz, compensating for wind loads and fast slewing maneuvers without skipping steps.
- **Bus Addressing:** The azimuth axis is addressed at `0xE0` and the elevation axis at `0xE1`, supporting relative and absolute positioning, velocity profiling, and hardware homing commands (`0x94`).

---

## Tracking Architecture:
{% include image-gallery.html images="tracker_pipeline.svg" height="520" %}

The tracking system operates across three main stages:
1. **Airborne Telemetry Downlink:** The flight computer streams real-time position, velocity, and barometric pressure data over 915 MHz LoRa.
2. **Ground Station Self-Localization:** The ground tracker determines its own position and heading using onboard GPS, an electronic compass, and an internal magnetic declination table to find true north without manual surveying.
3. **Coordinate Transformation & Steering:** The motion controller converts the rocket's coordinates into a local ENU frame using WGS-84 ellipsoid calculations, generating real-time azimuth and elevation commands for the stepper motors.

---

## Coordinate Math & Earth Curvature:
Over long flight distances and high altitudes, simple flat-Earth or spherical assumptions introduce pointing errors that can exceed the beamwidth of high-gain antennas. To maintain sub-degree accuracy, the tracking firmware transforms the rocket's GPS coordinates using the WGS-84 reference ellipsoid into a local ENU tangent plane centered at the ground station, resolving line-of-sight azimuth and elevation angles directly.

---

## GPS Outage Handling:
Commercial GPS receivers are subject to velocity and altitude lockouts (CoCom limits), and rocket vibration or exhaust plumes can cause temporary loss of satellite lock during boost. If GPS drops out during ascent, the flight software switches to inertial dead-reckoning using the onboard 6-axis IMU and Bosch BMP581 barometer. This maintains continuous trajectory estimates and keeps the ground antenna tracking smoothly through supersonic flight until GPS re-locks.

---

## True North Alignment:
To aim accurately without manual compass surveying at the pad, the tracker self-aligns using its onboard GPS, electronic compass, and a 2D World Magnetic Model (WMM) lookup table stored in firmware. By computing the local magnetic declination offset from its GPS coordinates, the system converts magnetic heading to true north to offset all motor azimuth commands. 

The firmware supports two tracking modes:
- **Static Mode (Launch Pad):** Filters GPS coordinates and compass readings before launch to remove multipath noise and establish a stable ground datum for the flight.
- **Dynamic Mode (Recovery Vehicle):** Continuously updates base station position and heading in real time when mounted to a chase vehicle, keeping antennas aimed at the payload while in motion.

---

## System Features:
- **Closed-Loop Motion:** Real-time UART control with magnetic encoder feedback to prevent skipped steps under wind loads.
- **Dedicated Power Distribution:** 1S Li-Ion battery powers the logic board, with a custom USB-C PD PCB supplying the stepper motors from an external power bank.
- **Self-Alignment:** Calibrates heading and location on boot using onboard GPS and electronic compass with WMM magnetic declination lookup.
- **Dual Antenna Tracking:** Simultaneously points 915 MHz LoRa and 5.8 GHz video antennas.
- **Ground Logging:** Logs incoming telemetry directly to 4GB eMMC storage while streaming live data over serial.

---

## Components:
- Central Controller: Duplicate [Rasalhague Avionics Board](/projects/rasalhague-telemetry/index/)
- Power Delivery: 1S Li-Ion for logic and custom USB-C PD power PCB from an external power bank for steppers
- RF Front-End: EBYTE E22-900M33S with Semtech SX1262 and 2W PA / LNA
- Stepper Motors: 2x NEMA 17 Stepper Motors for azimuth and elevation
- Closed-Loop Drivers: 2x Makerbase MKS SERVO42C Controllers
- Video Transmitter: RunCam WiFiLink 2 System
- Directional Antennas: Helical and Yagi Array
- Local Sensors: u-blox SAM-M10Q GNSS, BMP581 Barometer, MMC5983MA Compass
