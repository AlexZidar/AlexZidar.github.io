---
layout: post
title: Custom Flight Computer
description: As part of my work on Swamp Launch, I independently developed a barometric-altitude-based flight computer with an integrated accelerometer. This computer is designed for use in test flights conducted by the staging sub-team and will play a crucial role in the future development of staged-flight systems. The flight computer continuously records acceleration and filtered altitude data to a CSV file on the onboard microSD card. Altitude is also checked against a user-set altitude, at which point a servo is actuated to separate the stages via mechanical linkage. Absolute magnitude of acceleration is calculated and used for launch detection.
skills: 
  - Embedded Systems
  - Electrical Design
  - Avionics
  - Control Systems

main-image: /sonos.png
---

---
## Components:
###Teensy 4.1 Microcontroller
###BMP390 Barometric Pressure Sensor
###GY-521 Accelerometer & Gyroscope Module
###Connections For Power, Switch, and Servo

