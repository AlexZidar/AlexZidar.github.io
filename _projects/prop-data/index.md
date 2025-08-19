---
layout: post
title: Custom Data Acquisition System
description: As another contribution to Swamp Launch, I independently developed a custom data aquisition system (DAQ) for propulsion development. This DAQ is designed for use in testing and verifying prototype rocket motors and propellant formulation. The system records casing temperature, thrust, and combustion chamber pressure. This data is then immediately stored as a CSV to the on-board micro-SD card. Supplemental software was developed in Python for rapid report making and classification of rocket motors.

skills: 
  - C++
  - Python
  - Embedded Systems
  - Data Analytics
  - Propulsion
  - SolidWorks
  - 3D Printing


main-image: /daqlaunch.jpg
---

---


## Electronics:
{% include image-gallery.html images="daqelectronics.jpg" height="400" %}  

## Enclosure:
{% include image-gallery.html images="daqenclosure.jpg" height="400" %}  

## Analysis Tool:
{% include image-gallery.html images="daqanalysis.png" height="400" %}

## Example Output Report:
{% include image-gallery.html images="daqresults1.png" height="450" %}
{% include image-gallery.html images="daqresults2.png" height="400" %}

## Components:
- Teensy 4.1 Microcontroller
- Adafruit MCP9601 Thermocouple Amplifier
- LM085IT3.3 Linear Dropout Regulator
- SparkFun Qwiic Scale - NAU7802
- 1000 kG Load Cell
- 1000 psi Pressure Transducer
-	Generic K-Type Thermocouple
- Indicator LED
- Record Switch
- JST 1x2 Header
- JST 1X3 Header

## Capabilities:
- Temperature at 200hz
- Thrust at 320hz
- Pressure at 320hz
- Automatic Motor Classification and Report Generation



## Report Maker Code Snippet:
### Highlights   
```Python
void loop() {
  // Read the pulse width from the color control PWM input
  unsigned long colorPulseWidth = pulseIn(PWM_COLOR_PIN, HIGH, 30000);
  
  // Read the pulse width from the brightness control PWM input
  unsigned long brightnessPulseWidth = pulseIn(PWM_BRIGHTNESS_PIN, HIGH, 30000);
  
  // If no pulse is detected on the color input, skip this loop iteration
  if (colorPulseWidth == 0) {
    return;
  }
  
  // For debugging: print the measured pulse widths
  Serial.print("Color Pulse width: ");
  Serial.print(colorPulseWidth);
  Serial.print(" us, Brightness Pulse width: ");
  Serial.print(brightnessPulseWidth);
  Serial.println(" us");
   // Map the color pulse width to an RGB color.
  uint8_t r = 0, g = 0, b = 0;
  if (colorPulseWidth < 1100) {
    r = 255; g = 0;   b = 0;    // Red
  } else if (colorPulseWidth < 1200) {
    r = 255; g = 128; b = 0;    // Orange
  } else if (colorPulseWidth < 1300) {
    r = 255; g = 255; b = 0;    // Yellow
  } else if (colorPulseWidth < 1400) {
    r = 0;   g = 255; b = 0;    // Green
  } else if (colorPulseWidth < 1500) {
    r = 0;   g = 255; b = 255;  // Cyan
  } else if (colorPulseWidth < 1600) {
    r = 0;   g = 0;   b = 255;  // Blue
  } else if (colorPulseWidth < 1700) {
    r = 75;  g = 0;   b = 130;  // Indigo
  } else if (colorPulseWidth < 1800) {
    r = 148; g = 0;   b = 211;  // Violet
  } else {
    r = 255; g = 255; b = 255;  // White
  }
```
