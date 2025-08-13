---
layout: post
title: Custom Flight Computer
description: As part of my work on Swamp Launch, I independently developed a barometric-altitude-based flight computer with an integrated accelerometer. This computer is designed for use in test flights conducted by the staging sub-team and will play a crucial role in the future development of staged-flight systems. The flight computer continuously records acceleration and filtered altitude data to a CSV file on the onboard microSD card. Altitude is also checked against a user-set altitude, at which point a servo is actuated to separate the stages via mechanical linkage. Absolute magnitude of acceleration is calculated and used for launch detection.
skills: 
  - C++
  - Embedded Systems
  - Electrical Design
  - Avionics
  - Control Systems

main-image: /mainFC.jpg
---

---
## Components:
### Teensy 4.1 Microcontroller
### BMP390 Barometric Pressure Sensor
### GY-521 Accelerometer & Gyroscope Module
### Connections For Power, Switch, and Servo



## Final Product:
{% include image-gallery.html images="mainFC.jpg" height="400" %}



## Traces and Connections:
{% include image-gallery.html images="underFC.jpg" height="400" %}

## Pressure Sensor Calibration Snippet
### Accounts for local pressure changes and erroneous readings on startup.
```C++
void calibrateBMP() {
  // Discard initial readings
  for (int i = 0; i < BMP_WARMUP_READINGS; i++) {
    bmp.performReading();
    delay(100);
  }
  
  // Take multiple readings and average them for stability
  float altitudeSum = 0;
  int validReadings = 0;
  
  for (int i = 0; i < 10; i++) {
    if (bmp.performReading()) {
      // Use library's altitude calculation
      altitudeSum += bmp.readAltitude(1013.25);
      validReadings++;
    }
    delay(100);
  }
  
  if (validReadings > 0) {
    baselineAltitude = altitudeSum / validReadings;
    
    Serial.print("Baseline altitude: ");
    Serial.print(baselineAltitude);
    Serial.println(" m");
  } else {
    Serial.println("Calibration failed!");
    baselineAltitude = 0;
  }
}
```
