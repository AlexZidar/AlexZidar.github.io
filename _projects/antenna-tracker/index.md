---
layout: post
title: Autonomous Rocket Tracker & Ground Station
description: An autonomous 2-axis (pan-tilt) antenna tracker engineered for Swamp Launch Student Rocketry to track our high-powered rockets during high-speed ascent and recovery. By continuously aligning high-gain directional antennas with the rocket's position, the ground station maintains a robust, low-latency RF telemetry link and live-streamed onboard video feed throughout the flight. The tracker utilizes a duplicate Rasalhague telemetry board as its central receiver and real-time motion controller.
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
The ground tracker and rocket avionics operate as an integrated pair. The ground station uses an identical duplicate [**Rasalhague Telemetry Flight Computer**](/projects/rasalhague-telemetry/index/) as its central receiver and controller:
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

## Tracking Architecture & Geodesy Pipeline:
{% include image-gallery.html images="tracker_pipeline.svg" height="520" %}

The end-to-end tracking architecture spans three synchronized operational domains:
1. **Airborne State Estimation & Telemetry Downlink:** The flight computer continuously evaluates navigation solution validity, streaming high-rate position, velocity, and pressure packets over long-range 915 MHz LoRa.
2. **Autonomous Base Station Localization:** The ground receiver self-localizes in geodetic space, pairing an onboard GPS fix with a World Magnetic Model declination lookup table and electronic compass to resolve True Geodetic North without external survey tools.
3. **Ellipsoidal Kinematics & Closed-Loop Steering:** The central motion controller transforms raw spherical coordinates into a topocentric East-North-Up (ENU) tangent plane via WGS-84 oblate spheroid equations, calculating real-time azimuth and elevation steering commands for dual closed-loop stepper axes.

---

## Accounting for an Oblate Spheroid (Curved Earth Geodesy):
High-powered sounding rockets achieve altitudes exceeding 30,000–50,000+ feet and can drift miles downrange under high-altitude jet streams. Over these baselines, simplistic flat-Earth assumptions or spherical approximations introduce multiple degrees of angular pointing error. Because high-gain directional antennas (such as 14 dBi axial-mode helicals and multi-element Yagi arrays) exhibit narrow half-power beamwidths (18° – 25°), a pointing divergence of even 2° – 5° causes severe link margin degradation or outright signal loss.

To maintain sub-degree pointing accuracy, the tracking algorithm models the Earth as an **oblate spheroid** flattened at the poles by centrifugal planetary rotation, utilizing the **WGS-84 (World Geodetic System 1984)** reference ellipsoid:
- **Semi-major axis (equatorial radius):** `a = 6,378,137.0 m`
- **Flattening:** `f = 1 / 298.257223563`
- **First eccentricity squared:** `e² = 2f - f² ≈ 0.00669437999014`

### 1. Geodetic to Earth-Centered Earth-Fixed (ECEF) Conversion
Both the rocket's reported position `(φ_R, λ_R, h_R)` and the ground station's local position `(φ_GS, λ_GS, h_GS)` are first mapped into a 3D Cartesian ECEF frame. The radius of curvature in the prime vertical `N(φ)` accounts for the ellipsoidal equatorial bulge:

```text
N(φ) = a / √(1 - e² · sin²(φ))

X = (N(φ) + h) · cos(φ) · cos(λ)
Y = (N(φ) + h) · cos(φ) · sin(λ)
Z = (N(φ) · (1 - e²) + h) · sin(φ)
```

### 2. Topocentric East-North-Up (ENU) Transformation
The relative line-of-sight displacement vector `Δr_ECEF = r_rocket - r_GS` is rotated into the local tangent plane centered at the ground station's latitude `φ_GS` and longitude `λ_GS`:

```text
[ ΔE ]   [ -sin(λ)             cos(λ)            0      ] [ X_rocket - X_gs ]
[ ΔN ] = [ -sin(φ)·cos(λ)     -sin(φ)·sin(λ)     cos(φ) ] [ Y_rocket - Y_gs ]
[ ΔU ]   [  cos(φ)·cos(λ)      cos(φ)·sin(λ)     sin(φ) ] [ Z_rocket - Z_gs ]
```

### 3. Look Angle Resolution
From the topocentric `(ΔE, ΔN, ΔU)` vector, the true geodetic line-of-sight azimuth `α_true` and elevation `θ` are calculated directly:

```text
d_ground = √(ΔE² + ΔN²)

α_true   = atan2(ΔE, ΔN)
θ        = atan2(ΔU, d_ground)
```

By calculating look angles in the true ellipsoidal topocentric frame, geometric projection distortion is completely eliminated across any latitude or launch trajectory.

---

## CoCom Limit Mitigation & Supersonic Inertial Dead-Reckoning:
Commercial GNSS receivers (including the onboard u-blox SAM-M10Q) are legally subject to **CoCom Regulations** (and ITAR export controls), which mandate navigation lockouts if a vehicle exceeds velocities of 1,000 knots (514 m/s / Mach 1.5+) and/or altitudes of 18,000 meters (59,000 ft). Furthermore, extreme launch jerk, motor acoustic vibration, and ionized rocket exhaust plumes can induce loss of phase lock in GNSS carrier tracking loops during max-Q.

To prevent the ground station from losing antenna lock during supersonic boost, the airborne avionics architecture on [**Rasalhague**](/projects/rasalhague-telemetry/index/) implements an automated **inertial failover pipeline**:

- **Lockout Detection:** The flight computer polls the `UBX-NAV-PVT` navigation message at high frequency, monitoring fix validity flags (`gnssFixOK`), carrier-to-noise thresholds, and position dilution of precision (PDOP).
- **IMU Dead-Reckoning Failover:** If GPS lock drops or CoCom thresholds are triggered during boost, the state estimator automatically transfers control to high-rate strapdown inertial dead-reckoning. The system integrates 6-axis IMU linear accelerations (ST LSM6DSOX) through orientation quaternions `q`, initialized from the vehicle's last confirmed kinematic state vector:
  ```text
  v(t) = v(t - Δt) + (q ⊗ a_body ⊗ q* - g) · Δt
  p(t) = p(t - Δt) + v(t) · Δt
  ```
- **Barometric Altitude Anchoring:** Accelerometer double-integration suffers from quadratic drift (`~ t²`). To prevent runaway vertical error during the GPS outage, the vertical state is firmly anchored to the **Bosch BMP581 precision barometer**. Immune to CoCom velocity lockouts, the barometer delivers high-rate (100 Hz), drift-free pressure altitude through the entire boost phase.
- **Continuous Telemetry Stream:** The projected state vector is packaged into standard binary LoRa packets and broadcast down to the ground station. The ground gimbal continues tracking smoothly along the vehicle's true ascent arc through the supersonic window until GNSS tracking loops re-lock post-burnout.

---

## Autonomous True North Localization via Geomagnetic Declination Lookup:
To orient the pan-tilt gimbal's motorized end-effector toward the rocket, the ground station must know its own orientation relative to **True (Geographic) North**. 

The ground station's onboard **MMC5983MA 3-axis electronic compass** measures Earth's magnetic flux density vectors, determining heading relative to **Magnetic North**. However, Magnetic North and True North diverge by the **Magnetic Declination angle (δ)**, which varies substantially by geographic location (ranging from +15° East to -15° West across North America). Because high-gain helical and Yagi antennas have narrow beamwidths, a 5° – 10° uncorrected magnetic declination error would cause the antenna array to point completely off-target.

To eliminate the need for manual compass surveying or physical sighting during field operations, the ground station performs autonomous True North localization:

1. **Local GNSS Fix:** Upon boot, the ground receiver acquires its local geodetic coordinates `(φ_GS, λ_GS)`.
2. **World Magnetic Model (WMM) Lookup:** The ESP32-S3 queries an internal 2D **magnetic declination lookup table (LUT)** pre-computed from NOAA's World Magnetic Model. By bilinearly interpolating the LUT using local latitude and longitude, the controller computes the local declination offset `δ` to within `< 0.1°` without requiring cellular or internet access.
3. **Hard & Soft-Iron Calibration:** Raw magnetometer measurements are compensated for chassis hard-iron biases (static fields from screws, motor coils, and casing) and soft-iron ferromagnetic distortions:
   ```text
   B_cal = A_soft · (B_raw - B_hard)
   ```
4. **True North Resolution:** Magnetic heading `Ψ_mag = atan2(B_y,cal, B_x,cal)` is combined with the local declination:
   ```text
   Ψ_true = Ψ_mag + δ
   ```
5. **Gimbal Azimuth Coordinate Mapping:** The azimuth command sent to the pan stepper motor is offset by the ground chassis's True North heading:
   ```text
   α_motor = α_true - Ψ_true
   ```

### Dual Operating Modes:
- **Static Mode (Launch Pad Baseline):** During pre-launch countdown, the stationary ground station runs a 1 Hz multi-sample low-pass filter over GPS coordinates and magnetic heading. This eliminates high-frequency multipath noise and environmental magnetic jitter, locking in a high-confidence geodetic datum that is carried throughout the flight.
- **Dynamic Mode (Mobile Chase Vehicle):** If the tracker is deployed on the roof of a recovery vehicle to track the payload under canopy, the firmware continuously updates base GPS position and compass heading in real time. The gimbal dynamically compensates for the moving chase car's velocity and heading changes, maintaining antenna lock on the descending rocket.

---

## Real-Time Target Tracking Snippet:
### Computes WGS-84 ellipsoidal look angles and commands closed-loop steppers:
```C++
#include "stepper_controller.h"
#include "telemetry_format.h"
#include <cmath>

// WGS-84 Reference Ellipsoid Constants
constexpr double WGS84_A  = 6378137.0;            // Semi-major axis (meters)
constexpr double WGS84_F  = 1.0 / 298.257223563;  // Flattening
constexpr double WGS84_E2 = 2.0 * WGS84_F - WGS84_F * WGS84_F; // First eccentricity squared

// Convert Geodetic (lat/lon in radians, alt in meters) to ECEF (X, Y, Z)
void geodeticToECEF(double latRad, double lonRad, double altM, double& X, double& Y, double& Z) {
    double sinLat = sin(latRad);
    double cosLat = cos(latRad);
    double N = WGS84_A / sqrt(1.0 - WGS84_E2 * sinLat * sinLat);
    X = (N + altM) * cosLat * cos(lonRad);
    Y = (N + altM) * cosLat * sin(lonRad);
    Z = (N * (1.0 - WGS84_E2) + altM) * sinLat;
}

// Computes Azimuth and Elevation look angles from incoming rocket telemetry
void updateTrackerTarget(const TelemetryPacket& rocket, double gsLatDeg, double gsLonDeg, double gsAltM, float gsTrueNorthDeg) {
    double gsLatRad = gsLatDeg * DEG_TO_RAD;
    double gsLonRad = gsLonDeg * DEG_TO_RAD;
    double rkLatRad = rocket.gps_latitude_deg * DEG_TO_RAD;
    double rkLonRad = rocket.gps_longitude_deg * DEG_TO_RAD;

    // 1. Convert Ground Station and Rocket Geodetic positions to WGS-84 ECEF
    double gsX, gsY, gsZ;
    double rkX, rkY, rkZ;
    geodeticToECEF(gsLatRad, gsLonRad, gsAltM, gsX, gsY, gsZ);
    geodeticToECEF(rkLatRad, rkLonRad, rocket.altitude_msl_m, rkX, rkY, rkZ);

    // 2. Relative ECEF vector
    double dX = rkX - gsX;
    double dY = rkY - gsY;
    double dZ = rkZ - gsZ;

    // 3. Rotate into local topocentric East-North-Up (ENU) tangent plane
    double sinLat = sin(gsLatRad), cosLat = cos(gsLatRad);
    double sinLon = sin(gsLonRad), cosLon = cos(gsLonRad);

    double dEast  = -sinLon * dX + cosLon * dY;
    double dNorth = -sinLat * cosLon * dX - sinLat * sinLon * dY + cosLat * dZ;
    double dUp    =  cosLat * cosLon * dX + cosLat * sinLon * dY + sinLat * dZ;

    double groundDist = sqrt(dEast * dEast + dNorth * dNorth);

    // 4. Compute True Geodetic Look Angles
    float targetAzimuth = atan2(dEast, dNorth) * RAD_TO_DEG;
    if (targetAzimuth < 0.0f) targetAzimuth += 360.0f;

    float targetElevation = atan2(dUp, groundDist) * RAD_TO_DEG;
    targetElevation = fmax(0.0f, fmin(targetElevation, 90.0f));

    // 5. Offset Azimuth by Ground Station True North chassis heading
    float motorAzimuth = targetAzimuth - gsTrueNorthDeg;
    while (motorAzimuth < 0.0f)   motorAzimuth += 360.0f;
    while (motorAzimuth >= 360.0f) motorAzimuth -= 360.0f;

    // 6. Command closed-loop MKS SERVO42C stepper drivers over UART
    writeMotorAngle(MOTOR_ID_AZIMUTH, motorAzimuth, 1200);
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
- Central Controller: Duplicate [**Rasalhague Avionics Board**](/projects/rasalhague-telemetry/index/) (ESP32-S3-N8)
- RF Front-End: EBYTE E22-900M33S (Semtech SX1262 + 2W PA / LNA)
- Pan & Tilt Motors: 2x NEMA 17 High-Torque Stepper Motors
- Closed-Loop Drivers: 2x Makerbase MKS SERVO42C Controllers
- Video Transmitter: RunCam WiFiLink 2 System
- Directional Antennas: High-Gain Helical and Yagi-Uda Array
- Local Sensors: u-blox SAM-M10Q GNSS, BMP581 Barometer, MMC5983MA Compass
- Power System: 3S LiPo with onboard buck regulation
