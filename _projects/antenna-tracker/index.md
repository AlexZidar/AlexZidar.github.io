---
layout: post
title: Autonomous Rocket Tracker & Ground Station
description: An autonomous 2-axis (pan-tilt) antenna tracker engineered for Swamp Launch Student Rocketry to track our high-powered rockets during high-speed ascent and recovery. By continuously aligning high-gain directional antennas with the rocket's position, the ground station maintains a robust, low-latency RF telemetry link and live-streamed onboard video feed throughout the flight. The tracker utilizes a duplicate Rasalhague telemetry board as its central receiver and real-time motion controller.
skills: 
  - C++
  - Embedded Systems
  - Control Systems
  - Mechatronics
  - Python
  - SolidWorks
  - 3D Printing

main-image: /pan_tilt.gif
card-image: /pan_tilt_zoom.gif
order: 1
---

---
## Ascent Tracking & Flight Profile:
{% include image-gallery.html images="tracker_launch.jpg" height="400" %}  

---

## Overview:
During high-power rocket flights with **Swamp Launch Student Rocketry**, maintaining a high-bandwidth, reliable radio frequency (RF) link is critical for both real-time telemetry streaming and live onboard video. Standard omnidirectional antennas suffer from severe free-space path loss and multipath fading as the vehicle climbs thousands of feet and drifts downrange. 

To overcome this, I developed an autonomous 2-axis (pan and tilt) ground antenna tracker. The system dynamically aims high-gain directional antennas directly at the ascending rocket in real time, maximizing signal margin and ensuring continuous data reception from pad release to touchdown.

---

## Interactive 3D Ground Station CAD Model:
{% include model-viewer.html model="/_projects/antenna-tracker/Project_Sauron.glb" alt="Project Sauron: Autonomous Ground Station Gimbal CAD" height="520" pitch="-90" %}

---

## Hand-in-Hand Dual Board Architecture:
The ground tracker and rocket avionics operate as an integrated pair. The ground station uses an identical duplicate **Rasalhague** board as its central receiver and controller:
- **RF Demodulation:** The onboard Semtech SX1262 LoRa module receives binary telemetry packets broadcast by the rocket at 915 MHz.
- **Local Sensors & Reference:** The ground board utilizes its own onboard u-blox GPS, BMP581 barometer, and electronic compass to establish a precise local East-North-Up (ENU) coordinate frame without manual calibration.
- **Dual-Stream Link:** The tracker platform carries both high-gain directional LoRa antennas (for sensor telemetry) and directional 5.8 GHz video receiver antennas (for low-latency cockpit video).

---

## Live-Streamed Video Integration:
{% include image-gallery.html images="runcam_wifilink.png" height="380" %}

To deliver a pilot’s-eye view of the launch, the rocket carries a RunCam WiFiLink digital video transmitter in the avionics bay. The ground station's motorized gimbal keeps high-gain helical receiver antennas aimed at the airframe, preserving video bitrate and frame rates that would otherwise drop out during high-G flight regimes.

---

## Vehicle Integration:
{% include image-gallery.html images="rocket_field.jpg" height="400" %}

---

## Pan / Tilt Gimbal & Closed-Loop Motion:
- **Actuation:** 2x NEMA 17 stepper motors driving precision azimuth (pan) and elevation (tilt) axes.
- **Closed-Loop Drivers:** Makerbase **MKS SERVO42C** closed-loop stepper controllers communicate directly with the ESP32-S3 over a high-speed hardware UART bus (115200 baud).
- **Zero Step Loss:** Integrated magnetic encoders monitor shaft position at 20 kHz, automatically compensating for wind loads and high-acceleration tracking maneuvers without lost steps.
- **Native Addressing:** The pan axis is addressed at `0xE0` and the tilt axis at `0xE1`, supporting relative/absolute positioning, velocity profiling, and one-command hardware homing (`0x94`).

---

## Real-Time Target Tracking Snippet:
### Computes look angles from incoming rocket telemetry and commands closed-loop steppers:
```C++
#include "stepper_controller.h"
#include "telemetry_format.h"

// Calculate Azimuth and Elevation to rocket from Ground Station coordinates
void updateTrackerTarget(const TelemetryPacket& rocket, double groundLat, double groundLon, float groundAlt) {
    // Convert GPS delta to local flat-Earth ENU coordinates (meters)
    double dLat = (rocket.gps_latitude_deg - groundLat) * 111319.5;
    double dLon = (rocket.gps_longitude_deg - groundLon) * 111319.5 * cos(groundLat * DEG_TO_RAD);
    float dAlt = rocket.gps_alt_msl_m - groundAlt;
    
    float groundDistance = sqrt(dLon * dLon + dLat * dLat);
    
    // Compute target look angles (degrees)
    float targetAzimuth = atan2(dLon, dLat) * RAD_TO_DEG;
    if (targetAzimuth < 0.0f) targetAzimuth += 360.0f;
    
    float targetElevation = atan2(dAlt, groundDistance) * RAD_TO_DEG;
    targetElevation = constrain(targetElevation, 0.0f, 90.0f);
    
    // Command closed-loop MKS SERVO42C stepper drivers over UART
    writeMotorAngle(MOTOR_ID_AZIMUTH, targetAzimuth, 1200);
    writeMotorAngle(MOTOR_ID_ELEVATION, targetElevation, 800);
}
```

---

## System Capabilities:
- **Closed-Loop Stepper Control:** Real-time UART bus commanding up to 1200 pulses/sec with magnetic encoder position verification.
- **Autonomous Alignment:** Self-aligns heading and position using base-station GPS and MMC5983MA 3-axis electronic compass.
- **High-Gain Dual Link:** Simultaneously tracks 915 MHz LoRa telemetry and 5.8 GHz digital live video stream.
- **Redundant Ground Logging:** Logs all incoming telemetry directly to onboard 4GB eMMC storage while streaming CSV data to ground monitors over serial.

---

## Components:
- Central Controller: Duplicate Rasalhague Avionics Board (ESP32-S3-N8)
- RF Front-End: EBYTE E22-900M33S (Semtech SX1262 + 2W PA / LNA)
- Pan & Tilt Motors: 2x NEMA 17 High-Torque Stepper Motors
- Closed-Loop Drivers: 2x Makerbase MKS SERVO42C Controllers
- Video Transmitter: RunCam WiFiLink 2 System
- Directional Antennas: High-Gain Helical and Yagi-Uda Array
- Local Sensors: u-blox SAM-M10Q GNSS, BMP581 Barometer, MMC5983MA Compass
- Power System: 3S LiPo with onboard buck regulation
